# 第十四章 FTP、NFS、Samba 配置
## 一、vsftpd‑FTP服务（跨系统文件传输，端口21）
### 1. 安装与启动
```bash
dnf install -y vsftpd
systemctl start vsftpd
systemctl enable vsftpd
```
防火墙放行21端口
```bash
firewall-cmd --add-port=21/tcp --permanent
firewall-cmd --reload
```
2. 配置文件 `/etc/vsftpd/vsftpd.conf`常用配置
```bash   
vim /etc/vsftpd/vsftpd.conf
```
```ini
anonymous_enable=NO        # 关闭匿名用户（生产必须关闭）
local_enable=YES           # 允许本地系统用户登录
write_enable=YES           # 开启上传写入权限
local_umask=022            # 文件权限755
chroot_local_user=YES      # 将用户禁锢在家目录，不能跳出上级目录
allow_writeable_chroot=YES # chroot后允许写入
```
3. 创建 ftp 用户（系统用户）
```bash
useradd ftpuser -s /sbin/nologin  # 禁止登录系统，只用来ftp
passwd ftpuser
```

重启服务：
```bash
systemctl restart vsftpd
```
客户端用 FileZilla 连接：`服务器IP，端口21`。
>被动模式配置（客户端连不上时添加到 vsftpd.conf 末尾）
```ini
pasv_min_port=40000
pasv_max_port=40100
```
防火墙放行被动端口：
```bash
firewall-cmd --add-port=40000-40100/tcp --permanent
firewall-cmd --reload
```

## 二、NFS（Linux‑to‑Linux 共享）
NFS：仅 Linux 之间挂载，Windows 默认不支持。服务端：nfs‑utils，rpc‑bind。

### 1. 服务端配置（共享目录）
安装软件
```bash
dnf install -y nfs-utils rpcbind
systemctl start rpcbind nfs-server
systemctl enable rpcbind nfs-server
```

### 1.1 创建共享目录
```bash
mkdir -p /data/nfs
chmod 777 /data/nfs
```

### 1.2 编辑导出配置 /etc/exports
语法：共享目录 客户端IP(权限)
```bash
vim /etc/exports
```

写入下面一行
`/data/nfs 192.168.1.0/24(rw,sync,no_root_squash)`

参数解释：

   - rw：读写；ro只读
   - sync：同步写入磁盘；async 异步
   - no_root_squash：客户端 root 拥有服务端 root 权限（测试环境用，生产慎用）
   - root_squash：客户端 root 映射成 nobody（生产推荐）

生效配置
```bash
exportfs -r
# 查看发布的共享
exportfs -v
```

防火墙放行 NFS：
```bash
firewall-cmd --add-service=nfs --permanent
firewall-cmd --reload
```

### 2. 客户端（另一台 Linux 机器挂载）
```bash
dnf install -y nfs-utils
# 查看服务端共享目录
showmount -e 192.168.10.10
# 创建挂载点并挂载
mkdir /mnt/nfs
mount 192.168.10.10:/data/nfs /mnt/nfs
```
开机自动挂载写入 `/etc/fstab`：
```plaintext
192.168.10.10:/data/nfs   /mnt/nfs   nfs defaults 0 0
```

取消挂载：`umount /mnt/nfs`。


## 三、Samba：Linux 和 Windows 互相共享文件
Samba 实现 Windows 访问 Linux 目录。

### 1. 安装服务端
```bash
dnf install -y samba samba-client
systemctl start smb nmb
systemctl enable smb nmb
```
防火墙放行 samba：
```bash
firewall-cmd --add-service=samba --permanent
firewall-cmd --reload
```

### 2. 配置文件 `/etc/samba/smb.conf`
先备份原文件：
```bash
cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
```
在文件末尾追加共享配置：
```ini
[share]
    path = /data/samba
    browseable = yes
    writable = yes
    guest ok = no
    create mask = 0644
    directory mask = 0755

    [share]：Windows 看到的共享名；
    guest ok=no：关闭匿名访问，必须账号密码登录。
```

创建目录并修改归属：
```bash
mkdir -p /data/samba
chown root:root /data/samba
chmod 777 /data/samba
```

### 3. 创建 Samba 密码（重点：samba 必须单独密码，不能用系统密码）

用户必须是系统里已经存在的 Linux 用户。

```bash
useradd smbuser -s /sbin/nologin
# 设置samba密码
smbpasswd -a smbuser
```
输入两遍密码。
校验配置语法
```bash
testparm
```

出现 OK 代表配置没问题，重启 smb：
```bash
systemctl restart smb
```

### 4. Windows 访问
Win+R 输入：`\\192.168.10.10\share`，输入 smbuser 和刚刚设置的 samba 密码即可。

## 四、三者对比

|服务|协议端口|适用场景|
|---|---|---|
|vsftpd|TCP21（控制）、数据端口|跨网络，Windows/Linux 都能连；文件上传下载；需要账号密码|
|NFS|2049，RPC随机端口|Linux‑Linux 局域网传输，速度最快；Windows 兼容性差；基于 IP 授权|
|Samba|139、445 端口|Windows 访问 Linux 文件；企业办公环境；独立 samba 账号|