---
title: "Windows安装配置MySQL5.7"
date: "2017-07-18 10:56:21"
lastMod: "2017-07-18 10:56:21"
categories: ["it"]
tags: ["mysql", "windows"]
---

#### 下载，地址：https://dev.mysql.com/downloads/mysql/5.7.html#downloads

#### 解压至安装的位置

#### 配置环境变量：
1. MYSQL，目标：MySQL安装目录
2. 添加PATH，目标：%MySQL%\bin

#### 进入MySQL的bin目录，安装MySQL服务，使用**管理员**打开命令行，执行：
```bat
cd %MYSQL%\bin # 这一步是否有必要，未验证
mysqld -install
```

#### 添加配置文件：my.ini（未验证，未使用配置）

#### 招行初始化：
```bat
mysqld --initialize
```
稍等一会儿

#### 启动MySQL服务，将自动进行初始化

#### 停止MySQL服务

#### 手动运行，开启无密码的 MySQL Server：
```bat
mysqld --skip-grant-tables
```

#### 打开新命令行，登录MySQL，修改root密码
```bat
mysql -u root
grant all privileges on *.* to 'root'@'localhost' identified by '<新密码>' with grant option;
flush privileges;
exit
```

#### 退出MySQL
可能需要在任务管理器中手动结束mysqld进程

#### 启动MySQL服务，完成