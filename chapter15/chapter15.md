# 第十五章 综合项目实践——基于 openEuler 部署 LNMP 环境

## 一、什么是LNMP
### 1.1 LNMP 定义
LNMP 是 Linux + Nginx + MySQL/MariaDB + PHP 的缩写，是一套运行 PHP 动态网站的标准服务器环境栈。
四个字母分别对应四层组件，缺一不可，共同支撑带数据库交互的网页、后台系统、小程序后端。
你现在用的 openEuler 属于 Linux，正好适配这套环境。

1. L = Linux（操作系统，底层基石）
代表 Linux 类操作系统：openEuler、CentOS、Ubuntu、Debian 等。

   - 作用：所有软件（Nginx、数据库、PHP）都要运行在 Linux 系统之上，管理硬件、进程、文件、网络权限。
   - 为什么不用 Windows：服务器 Linux 占用资源更低、稳定性高、长期不关机、安全漏洞更少，web 服务器行业标准。

2. N = Nginx（Web 网页服务器，入口网关）
是接收用户访问的第一道程序。
核心功能：

   - 监听 80（http）、443（https）端口，接收浏览器发来的网址请求；
   - 处理静态资源：html、图片、css、js、视频，直接返回给浏览器，速度极快；
   - 转发动态 PHP 请求：遇到 .php 文件，不会自己解析，交给 PHP-FPM 处理；
   - 负载均衡、反向代理、防盗链、限流、SSL 加密、伪静态、多站点管理。

对比老方案 Apache
Nginx 高并发强，上万人同时访问也不容易卡顿，现在几乎所有网站都在用。

3. M = MySQL / MariaDB（关系型数据库，数据仓库）
M 原本指 MySQL，现在开源系统普遍使用 MariaDB（MySQL 官方分支，完全兼容、免费无版权限制）。
核心作用：永久存储网站所有动态数据
网站所有会变、需要保存的数据全部存在这里：

   - 用户：账号、密码、头像、会员等级
   - 内容：文章、评论、商品、订单、留言
   - 配置：网站设置、分类、权限

没有数据库会怎样？
只能打开静态 html，无法注册登录、无法提交表单、刷新页面数据全部消失。

4. P = PHP（动态脚本语言，业务逻辑处理器）
PHP 专门用来写网页交互逻辑，搭配数据库使用。
Nginx 本身看不懂 PHP 代码，依靠配套程序 php-fpm 运行 PHP。
PHP 能干的事：

   - 接收用户表单：登录、注册、留言、下单；
   - 连接数据库：查询、新增、修改、删除数据；
   - 动态生成页面：根据不同用户展示不同内容；
   - 文件上传、图片处理、接口开发（给小程序 / APP 提供数据）。

### 1.2 完整访问流程
举个例子：浏览器访问 http://服务器IP/test.php

   - 用户输入网址，网络请求到达服务器，先交给 Nginx；
   - Nginx 判断文件后缀是 .php，自己处理不了动态代码；
   - Nginx 通过 9000 端口把请求转发给 PHP-FPM；
   - PHP 代码执行，代码里有查询用户信息逻辑，于是连接 MariaDB；
   - 数据库查询完成，把数据返回给 PHP；
   - PHP 将数据拼接成完整网页代码，回传给 Nginx；
   - Nginx 把最终网页发给浏览器，页面展示出来。

补充：如果访问 index.html，Nginx 直接读取文件返回，不需要 PHP 和数据库参与。

### 1.3 LNMP 整体分层结构

    底层层：Linux（openEuler）管理硬件、网络、文件权限
    网关层：Nginx 接收访问、静态资源、请求分发
    逻辑层：PHP-FPM + PHP 处理网站业务逻辑
    存储层：MariaDB 持久化存储网站数据

### 1.4 LNMP 的优缺点
优点

   - 全开源免费，无商业授权费用；
   - Nginx 性能强悍，支持大量并发访问；
   - 资源占用低，低配服务器也能稳定运行；
   - PHP 上手简单，开发网站成本低；
   - 适配几乎所有 PHP 开源建站程序；
   - Linux 系统长期稳定，可常年不重启。

缺点

   - 仅支持 PHP 项目，不能直接运行 Java、Python、Go 程序；
   - 高大型分布式业务，原生 LNMP 架构需要额外做集群优化；
   - PHP 不适合超高运算量服务（适合网页、轻量接口）。

### 1.5 补充关键配套组件：PHP-FPM
很多人分不清 PHP 和 PHP-FPM：

   - PHP：编程语言本身；
   - PHP-FPM：PHP 的进程管理程序，负责监听 Nginx 转发的请求，调度进程执行 PHP 代码。

LNMP 环境必须启动 php-fpm 服务，否则无法解析 PHP 文件，访问 .php 会直接下载源码。


## 二、安装 PHP、Nginx

1. 安装Nginx
```bash
yum install nginx -y
```

2. 启用 PHP 模块并安装（以 PHP8.1 为例）
```bash
# 启用php81模块
dnf module enable php:8.1 -y

# 安装php核心+nginx扩展+数据库扩展
dnf install php-fpm php-cli php-mysqlnd php-gd php-mbstring php-curl php-xml php-opcache php-fileinfo -y
```

3. 启动 php-fpm 并开机自启
```bash
systemctl enable --now php-fpm
```

4. 调整 php-fpm 用户组（关键，Nginx 才能读写 PHP 文件）
编辑配置：
```bash
vim /etc/php-fpm.d/www.conf
```

修改两行：
```ini
user = nginx
group = nginx
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
```

重启 php-fpm 生效：
```bash
systemctl restart php-fpm
```

## 三、Nginx 配置解析 PHP 站点
1. 创建测试站点目录
```bash
mkdir -p /home/www/default
chown nginx:nginx /home/www/default
```

2. 创建 Nginx 站点配置
```bash
vim /etc/nginx/conf.d/default.conf
```

写入配置：
```nginx

server {
    listen 80;
    server_name _;
    root /home/www/default;
    index index.html index.htm index.php;

    # PHP解析
    location ~ \.php$ {
        fastcgi_pass unix:/run/php-fpm/www.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # 伪静态（可选，适配TP/WordPress）
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
}
```

3. 检查 Nginx 配置 + 重载
```bash
# 校验语法
nginx -t
# 重载配置
systemctl reload nginx
```

## 四、PHP 探针测试
```bash
echo "<?php phpinfo(); ?>" > /home/www/default/info.php
```

浏览器访问 http://服务器IP/info.php，能正常展示 PHP 信息代表 LNMP 打通。

## 五、生产环境安全配置补充

    防火墙放行 80/3306（不关闭防火墙）

```bash
firewalld --add-service=http --permanent
firewalld --add-service=https --permanent
firewalld --reload
```

php.ini 关闭危险函数、限制上传大小

```bash
vim /etc/php.ini
```
可根据要求调整：upload_max_filesize、post_max_size、disable_functions

## 六、如何使用LNMP
1. 刚才我们设置的网站根目录为 `/home/www/default`

```bash
cd /home/www/default
```

2. 创建并写入网页文件
```bash
vim index.html
```

3. 修改文件权限（Nginx 能正常读取）
```bash
chown nginx:nginx index.html
chmod 644 index.html
```

4. 访问网页
现在可以用虚拟机的IP地址在浏览器直接访问。
如果想要用物理机IP地址访问的话，需要配置vmware的端口转发

就能看到你写的网页。

5. 常用服务管理命令
```bash
# Nginx
systemctl start/stop/restart/status nginx

# PHP-FPM
systemctl start/stop/restart/status php-fpm

# MariaDB
systemctl start/stop/restart/status mariadb
```
