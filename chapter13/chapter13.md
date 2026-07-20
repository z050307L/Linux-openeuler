# 第十三章 MariaDB的安装与基本操作

## 1.安装mariadb

1）安装 mariadb‑server
```bash
dnf install -y mariadb-server mariadb
```

2）启动并设置开机自启
```bash
# 启动服务
systemctl start mariadb

# 设置开机自启
systemctl enable mariadb

# 查看运行状态
systemctl status mariadb
```
看到 `active (running) `代表启动成功。


3）安全初始化（必须执行，设置 root 密码、移除匿名用户）
```bash
mysql_secure_installation
```
选项参考：
- Enter current password for root (enter for none): 回车（初始无密码）
- Set root password? [Y/n]：y
- 输入你的 root 数据库密码；
- Remove anonymous users? [Y/n]：y
- Disallow root login remotely?：

        如果只本地访问：y；需要远程连接填 n

- Remove test database and access to it? [Y/n]：y
- Reload privilege tables now? [Y/n]：y

登录数据库测试
```bash
mariadb
```

## 2.开放远程访问（如需外部连接）
1）登录 mariadb 执行授权
```sql
# 允许root从任意IP访问
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '你的密码';
FLUSH PRIVILEGES;
exit;
```

2） 防火墙放行 3306 端口
```bash
firewall-cmd --add-port=3306/tcp --permanent
firewall-cmd --reload
```

## 3.常用命令
1）数据库操作命令
1. 查看所有数据库
```sql
SHOW DATABASES;
```

2. 创建数据库
语法：CREATE DATABASE 库名 [选项];
```sql
CREATE DATABASE testdb DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

3. 使用数据库
```sql
USE testdb;
```

4. 查看当前正在使用的库
```sql
SELECT DATABASE();
```

5. 删除数据库
```sql
DROP DATABASE IF EXISTS testdb;
```

2）用户管理
1. 创建用户
格式：'用户名'@'允许登录地址'

   - 'user'@'localhost'：只能本机登录
   - 'user'@'%'：任意远程 IP 都能登录

```sql
-- 创建用户 testuser，密码 123456，允许任意IP访问
CREATE USER 'testuser'@'%' IDENTIFIED BY '123456';

-- 仅本地登录
CREATE USER 'testuser'@'localhost' IDENTIFIED BY '123456';
```
>注意：MariaDB 密码尽量复杂，特殊符号建议用引号包裹。

2. 给用户授权
（1）给用户授予 testdb 库下全部权限
```sql
GRANT ALL PRIVILEGES ON testdb.* TO 'testuser'@'%';
```
`testdb.*：testdb 库里面所有表；*.* 代表所有库所有表`

（2）精细化授权（只给查询和插入权限）
```sql
GRANT SELECT,INSERT,UPDATE ON testdb.* TO 'testuser'@'%';
```

3. 刷新权限（授权后必须执行）
```sql
FLUSH PRIVILEGES;
```

4. 查看用户权限
```sql
SHOW GRANTS FOR 'testuser'@'%';
```

5. 修改用户密码
```sql
SET PASSWORD FOR 'testuser'@'%' = PASSWORD('NewPass@123');
FLUSH PRIVILEGES;
```

6. 删除用户
```sql
DROP USER IF EXISTS 'testuser'@'%';
```

7. 查询数据库里所有用户
```sql
SELECT user,host FROM mysql.user;
```
3）数据表操作（建表、增删改查）
1. 建表语句示例
```sql
USE testdb;
CREATE TABLE student(
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '主键自增',
    name VARCHAR(50) NOT NULL COMMENT '姓名',
    age INT DEFAULT NULL COMMENT '年龄',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='学生表';
```
- ENGINE=InnoDB：支持事务；MariaDB 默认引擎。

2. 查看当前库里面的表
```sql
SHOW TABLES;
```

3. 查看表结构
```sql
DESC student;
-- 或者详细查看建表语句
SHOW CREATE TABLE student;
```

4. 添加数据
```sql
INSERT INTO student(name,age) VALUES ('张三',20);
```

5. 查询数据
```sql
SELECT * FROM student;
SELECT id,name FROM student WHERE age>18;
```

6. 更新数据
```sql
UPDATE student SET age=21 WHERE id=1;
```

7. 删除数据
```sql
DELETE FROM student WHERE id=1;
```

8. 修改表结构
```sql
-- 添加字段
ALTER TABLE student ADD COLUMN email VARCHAR(100);
-- 删除字段
ALTER TABLE student DROP COLUMN email;
-- 修改字段类型
ALTER TABLE student MODIFY COLUMN age TINYINT;
```

9. 删除数据表
```sql
DROP TABLE IF EXISTS student;
```

4）MariaDB 外部命令（系统 shell 命令，不在 mysql 内部）
```bash
# 查看版本
mysql --version

# 导出整个数据库（shell环境执行）
mysqldump -uroot -p testdb > testdb_back.sql

# 导入sql文件
mysql -uroot -p testdb < testdb_back.sql

# 重启服务
systemctl restart mariadb
systemctl enable mariadb
```

## 4.配置文件路径
主配置文件：/etc/my.cnf
数据目录：/var/lib/mysql
日志目录：/var/log/mariadb