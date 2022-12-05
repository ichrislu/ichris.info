---
title: "CentOS安装mysql5.7"
date: "2017-07-20 16:40:00"
lastMod: "2017-07-20 16:40:00"
categories: ["it"]
tags: ["CentOS", "mysql"]
---

### 安装
#### 下载MySQL yum源安装包：
https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm

#### 安装MySQL yum源：
```shell
yum localinstall mysql57-community-release-el7-11.noarch.rpm
```

#### 安装
```shell
yum install mysql-community-server
```

### 配置
#### 编辑/etc/my.cnf文件，添加一行配置
```properties
skip-grant-tables = 1
```

#### 启动服务
```shell
systemctl start mysqld
```

#### 连接至MySQL服务
因为配置了跳过密码要求，输入mysql -u -p之后回车，提示输入密码的时候继续回车就可以连接至mysql了
修改root密码：
```shell
SET PASSWORD = PASSWORD('your new password');
ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
flush privileges;
```

#### 删除第1步中添加的配置
#### 重启服务

### 初始化
#### 连接
```shell
mysql -uroot -p # 输入root密码
```

#### 创建数据库
```sql
CREATE DATABASE `<数据库名>` CHARACTER SET 'utf8mb4' COLLATE 'utf8mb4_unicode_ci';
```

#### 创建用户
```sql
CREATE USER `<用户名>`@`%` IDENTIFIED BY '<密码>';
```

#### 添加授权(演示为ALL，更多权限请自己上网查）
```
GRANT ALL ON `<数据库名>`.* TO `<用户名>`@`%`;
```