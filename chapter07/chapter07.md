# 第七章 软件包管理

## 一、基础概念
1. **底层格式：RPM**
openEuler软件包后缀为`.rpm`；rpm是底层软件包格式。
- `rpm`命令：直接操作本地rpm安装包，**不会自动处理依赖**，依赖缺失时安装失败。
- `dnf`（Dandified YUM）：openEuler默认高级包管理器，yum升级版，自动解析依赖、下载安装，官方推荐使用；执行dnf命令必须为root用户。
> Ubuntu使用apt‑deb体系；openEuler、CentOS采用dnf‑rpm体系。

2. 软件仓库（repo）
仓库配置文件目录：`/etc/yum.repos.d/`，里面存放`.repo`文件；
全局配置文件：`/etc/dnf/dnf.conf`。
- 网络源：互联网下载软件包（官方源、华为镜像源）；
- 本地源：系统镜像文件作为安装源（断网环境使用）。

## 二、dnf核心常用命令（root执行）
|命令|功能|示例|
|----|----|----|
|`dnf repolist`|查看启用的软件仓库|`dnf repolist`|
|`dnf makecache`|刷新软件源缓存（更换源后执行）|`dnf makecache`|
|`dnf search 软件名`|搜索仓库里对应的软件|`dnf search vim`|
|`dnf install -y 软件名`|安装软件，‑y自动确认，无需手动输入y|`dnf install -y nano`|
|`dnf update`|1.不带参数：更新系统全部软件和内核<br>2.指定软件：仅升级单个程序|`dnf update -y` <br> `dnf update nginx`|
|`dnf remove 软件名`|卸载软件，自动移除无用依赖|`dnf remove nginx`|
|`dnf list installed`|查看系统已经安装的软件包|`dnf list installed`|
|`dnf info 软件名`|查看软件详细信息|`dnf info vim`|
|`dnf autoremove`|清理安装软件残留的无用依赖包|`dnf autoremove -y`|

### 补充参数说明
- `-y`：全部交互选项默认选yes，脚本里必用；
- `dnf group install "Development Tools"`：安装软件包组（开发工具集）。

## 三、rpm命令（底层命令）
> rpm只处理本地下载好的rpm文件，**不能自动解决依赖问题**。
```bash
rpm -ivh  xxx.rpm     #安装rpm包；i安装，v显示过程，h进度条
rpm -qa | grep 软件名 #查询软件是否安装
rpm -e 软件名         #卸载软件（容易出现依赖报错）
```
折中方案：dnf localinstall xxx.rpm，安装本地 rpm 并且自动下载依赖包

## 四、源码编译安装
### 1.概念
1. rpm、dnf安装的是编译好的二进制包，安装简单，但版本由软件仓库决定；
2. 源码安装：下载软件源代码（`.tar.gz`压缩包），在本机经过编译生成程序，可以自主选择软件版本、自定义功能；
3. 缺点：不会自动解决依赖；卸载不方便；必须提前安装编译环境。

### 必备编译环境（root执行，openEuler安装开发工具组）
```bash
dnf group install -y "Development Tools"
#包含gcc、make编译器
```
### 2. 源码安装标准 4 步骤
- 步骤 1：解压源码包
```bash
tar -zxvf 软件名.tar.gz
cd 解压后的目录
```

- 步骤 2：./configure 配置
作用：
  - 检查系统依赖环境是否齐全；缺少依赖就报错；
  - 指定安装路径（默认/usr/local/软件名）；开启或者关闭功能模块；
```bash
./configure --prefix=/usr/local/nginx
#--prefix 指定安装目录
```

- 步骤 3：make 编译
调用 gcc 编译器把源代码翻译成可执行程序；只是编译，还没有写入系统。
```bash
make
```

- 步骤 4：make install 安装
把编译好的文件复制到--prefix指定目录，完成安装。
```bash
make install
```

### 3. 卸载源码安装的软件
源码方式没有卸载命令，不能用 dnf remove：

1.如果记得安装目录：直接删除安装文件夹：

```bash
rm -rf /usr/local/nginx
```
2.部分软件支持：make uninstall。


## 五.三种安装方式对比
| 安装方式 | 命令 | 优点 | 缺点 |
| ---- | ---- | ---- | ---- |
| RPM | rpm‑ivh | 安装快速；仅本地包 | 依赖不会自动解决 |
| DNF | dnf install | 自动解决依赖，操作简单 | 版本受仓库限制 |
| 源码编译 | ./configure→make→make install | 版本自选，可以自定义功能 | 需要手动装依赖，卸载麻烦 |
