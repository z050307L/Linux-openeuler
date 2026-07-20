# 第十二章 Web服务器

## 1.Nginx or Apache

### 1.1基础概况

- Apache：老牌 web 服务器，全称 httpd，CentOS 软件包名字叫 httpd。
    采用多进程‑多线程模型，一个连接对应一个线程。

- Nginx：轻量级，事件驱动（epoll）、单进程多异步非阻塞模型。

二、核心区别
1. 并发能力（最大差异）

   - Apache：每来一个请求就分配 1 个线程，高并发下线程过多，CPU 上下文切换频繁，上万并发性能很差。适合并发量不大的网站。
   - Nginx：异步非阻塞、epoll 模型，一个工作进程可以处理成千上万连接，消耗内存极低；高并发、高连接场景碾压 Apache；互联网公司主流选择。

2. IO 模型

   - Apache：同步阻塞 IO；
   - Nginx：异步非阻塞 IO。

3. 模块扩展

   - Apache：模块加载方式：动态加载 / 静态编译，模块丰富稳定，但开启过多模块臃肿。
   - Nginx：模块编译时添加（默认不支持动态模块，新版 1.9.10 以后支持动态模块），第三方模块少于 Apache。

4. 稳定性

   - Apache：多进程模式，某个线程崩溃只影响当前连接，主进程不受影响，长期运行稳定。
   - Nginx：Worker 进程一旦崩溃该进程下所有连接断开；master 进程依旧正常。只要配置合理稳定性没问题。

5. 功能侧重
- Apache 优点：

    1. .htaccess 目录级配置（虚拟目录权限、防盗链、URL 重写），目录单独配置非常灵活；虚拟主机配置简单。
    2. 对 PHP 解析兼容性更好，搭配 mod_php 模块。
    3. 动态网站老牌首选，老式 PHP‑Apache 架构。

- Nginx 优点：

    1. 反向代理、负载均衡、动静分离、缓存、限流、SSL 性能极强；
    2. 静态文件 (html、图片、js) 处理速度远超 Apache；
    3. 可以做前端代理服务器，后端连接 Apache、Tomcat。

经典架构：Nginx（前端） + Apache/Tomcat（后端）。

6. 资源占用
同样 1000 并发：
   - Apache：占用几百 MB 内存；
   - Nginx：几十 MB 内存。

这章我们的主要目的是部署一个静态网站，所以我们选择Docker + Nginx
将nginx 跑在 docker 容器中。

## 2.前期准备

### 2.1 安装docker
```bash
dnf install -y docker
systemctl start docker
```
可以用
```bash
docker --version
```
来查看有没有安装成功

### 2.2 导入nginx镜像
如果网络允许的话，可以直接从外网下载nginx

```bash
docker pull nginx
```
如果网络不允许，需要导入nginx的镜像包
![](./picture/nginx.png)
cd到导入的目录下,再运行
```bash
sudo docker load -i nginx-alpine.tar
```
我这里导入到了/home/a1/下载，

```bash
cd /home/a1/下载
sudo docker load -i nginx-alpine.tar
```

完成后可以用
```bash
docker images
```
来查看有没有导入成功


### 2.3 设置网络和防火墙
1）如果虚拟机网络用的是桥接模式，可以不用设置端口转发。
如果虚拟机网络用的是NAT模式，需要设置端口转发：让外部设备通过指定端口访问内网设备服务的技术
![](./picture/wang.png)

>需要注意的是，如果用的是DHCP，每一次启动虚拟机，DHCP都会分配不同的IP地址，端口转发处的IP地址也需要对应更换。不想更换的可以参考前面的内容，设置静态IP

2）告诉Windows 防火墙：允许外部访问主机的 8080 端口，否则端口转发就没法生效。
以管理员身份运行windows的powershell，并输入如下命令
```bash
netsh advfirewall firewall add rule name="Kali8080" dir=in action=allow protocol=TCP localport=8080
```
3）同时让openeuler放行8080端口
```bash
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --reload
```

### 2.4 准备一个静态网站
创建一个工程文件夹，在里头写一个网站的html文件

我的是在/home/a1/桌面/MySite下的index.html
![](./picture/site.png)

## 3.部署静态网站
所有准备工作做好之后，打开终端，输入
```bash
docker run -d \
--name mysite \
-p 8080:80 \
-v /home/a1/桌面/MySite:/usr/share/nginx/html \
nginx:alpine
```
启动服务

### 3.1 代码详解
docker run：Docker 创建并启动一个新容器。

\：shell 里的换行符，只是为了命令分行写，便于阅读，一行写完可以去掉反斜杠。
下面逐个参数拆解：
1. -d
全称：--detach
   - 含义：后台守护模式运行容器
   - 作用：容器启动后放到后台运行，终端不会被占用；
    如果去掉 -d，nginx 日志会直接输出在终端，关闭终端网站就停止，生产环境必加 -d。

2. --name mysite
    --name：给容器自定义名称，如果不指定，docker 会随机生成名字。
    mysite：你的容器名字，之后所有操作（停止、重启、查看日志）都用这个名字，例如：
```bash
    docker stop mysite
    docker restart mysite。
```
    注意：同一个名字只能有一个容器，再次执行 run 命令会报错，必须先docker rm -f mysite删除旧容器。

3. -p 8080:80（端口映射）
格式：宿主机端口:容器内部端口

   - 后面的 80：是 nginx‑alpine 容器内部 Nginx 服务默认监听端口（Nginx 默认固定 80 端口，不能改）；
   - 前面的 8080：你的 openEuler 虚拟机对外开放的端口；

数据访问流向：
Windows 浏览器 → VMware NAT 端口转发 (8080) → openEuler 虚拟机的 8080 端口 → 容器内部的 80 端口 → Nginx 服务。

        容器内部端口：80（固定写死）；
        openEuler 虚拟机端口：8080；
        Windows 访问端口：8080（VM‑NAT 映射的端口）。

4. -v /home/a1/桌面/MySite:/usr/share/nginx/html（数据挂载）
   
    - -v = volume 数据卷挂载，
    格式：宿主机目录:容器内目录

    - 宿主机目录：/home/a1/桌面/MySite
    openEuler 虚拟机里存放网页的文件夹，index.html就放在这里，是物理真实目录；
    - 容器内目录：/usr/share/nginx/html
    nginx‑alpine 镜像默认读取网页的目录，Nginx 只会读取这个路径下的 html 文件；

挂载的特点：

   - 双向实时同步：你在 /home/a1/桌面/MySite 修改、新增网页文件，容器里立刻生效，不需要重启 docker 容器；
   - 容器删除后（docker rm mysite），容器内部文件全部清空，但是宿主机MySite文件夹和你的网页文件完全保留，不会丢失；
   - 权限问题：容器里 nginx 运行用户 uid=101，如果宿主机文件夹权限不足，就会出现 403 Forbidden，也就是你刚才遇到的问题。

5. nginx:alpine（最后一个参数：镜像名称）

    - nginx：镜像名字；
    - alpine：镜像标签，基于 alpine‑Linux（极简版 Linux）；
    - 对比：普通 nginx 镜像几百 MB，nginx:alpine只有二三十 M，体积小漏洞更少，非常适合部署静态网站；

如果你是离线导入镜像，docker images查看镜像名字标签必须和这里一致，不然会报错找不到镜像。

>启动完服务之后，主机在浏览器输入localhost:8080，即可访问网页
其他人想要访问需要输入虚拟机IP + :8080


## 4.KVM和VMware-workstation的区别

### 4.1 基础信息对比

|对比项|KVM‑QEMU（Linux 宿主机）|VMware Workstation Pro|
|---|---|---|
|虚拟化类型|内核级（Linux 内核模块，虚拟机作为 Linux 进程）|标准 Type‑2 应用程序，跑在操作系统之上|
|可用平台|仅 Linux 主机原生支持；Windows 不能原生 KVM|Windows、Linux 双平台都可以安装|
|组件构成|kvm 内核模块 + QEMU + libvirt + virt‑manager|VMware 自研 VMM 内核模块 + vmware‑tools|

### 4.2 性能差异

CPU
   - KVM：虚拟机被 Linux 内核原生调度，CPU 开销更低；大数量虚拟机并发、Linux 虚拟机场景性能更强。
   - Workstation：VMware 自研调度器；Windows 虚拟机优化更好；但是宿主机 OS 还要占用一部分资源，整体 CPU 损耗略高于 KVM。

磁盘 & 网络 I/O
   - KVM 依靠virtio‑blk、virtio‑net半虚拟化驱动，I/O 路径短，Linux 虚拟机磁盘和网络速度普遍优于 Workstation。
   - VMware‑Tools 针对 Windows Guest 优化到位，Windows 虚拟机里文件拷贝、磁盘读写表现更好。

3D 图形加速
   - Workstation：开箱即用支持 DX11、OpenGL，3D 加速简单稳定，做 Windows 里运行 CAD、模拟器兼容性更好；开启简单。
   - KVM：默认 3D 加速配置繁琐；GPU 硬件直通（PCI‑Passthrough）是 KVM 巨大优势，IOMMU 开启后可以把整块独显给虚拟机，性能接近真机；但配置复杂，Windows 宿主机做不到。


### 4.3 功能层面对比（桌面场景）
VMware Workstation 优势

   - 易用性极高：图形界面成熟，新建虚拟机、快照、克隆、共享文件夹、拖拽文件、Unity 无缝模式（虚拟机程序直接出现在宿主机桌面）开箱即用。
   - 快照管理友好：快照链、快照注释、克隆、链接克隆操作可视化；可以连接 ESXi 服务器上传下载虚拟机。
   - USB 设备兼容优秀：打印机、加密狗、外设即插即用；嵌套虚拟化打开简单，勾选选项即可跑 ESXi、KVM。
   - 虚拟网络模式齐全：桥接、仅主机、NAT、自定义虚拟交换机、端口转发图形界面配置，新手友好。

KVM‑QEMU（libvirt+virt‑manager）优势

   - 高度灵活、可自动化：virsh命令行完整，适合写 Shell 脚本批量创建、启动、快照；可以对接 OpenStack、Proxmox‑VE 向云环境过渡。Workstation 命令行工具功能偏弱，自动化很难。
   - 高级硬件直通：PCI‑GPU 直通、网卡直通、SR‑IOV、NUMA 绑核、HugePage 大页，这些企业级特性 KVM 原生支持；Workstation 桌面版不支持完整 PCI‑Passthrough，只能用有限虚拟显卡。
   - 磁盘格式选择多：qcow2 高级特性丰富（稀疏磁盘、内部快照、差分镜像），存储灵活性更强。
   - 无软件版权问题，大规模部署不需要花钱。

KVM 缺点（桌面环境痛点）

   - virt‑manager 界面简陋；高级配置（CPU 拓扑、内存热插拔、IOMMU 直通必须修改配置文件，门槛高）。
   - Windows‑KVM 的 virtio 驱动需要手动下载安装；共享文件夹配置麻烦，不能直接拖拽文件。
   - USB 设备支持弱，加密狗、工业外设经常识别失败；嵌套虚拟化配置繁琐。
   - Windows 系统完全不能原生跑 KVM，这是最大短板。

### 4.4 生态和扩展场景
KVM：

适合：Linux 运维学习、服务器环境模拟、后期学习 OpenStack、Proxmox‑VE、信创环境、大批量虚拟机自动化部署；宿主机是 Linux 优先选择。

局限：Windows 主机无缘原生 KVM；面向普通桌面用户过于硬核。

VMware Workstation：

适合：Windows 开发者、日常学习测试、Windows‑Linux 混合环境、经常嵌套虚拟化、连接 ESXi 环境。

局限：Pro 版收费；不能做大规模集群；无法对接云原生；后期往私有云升级还是要改用 KVM 体系。
