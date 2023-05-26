---
title: ubuntu 中使用 apt 安装 MySQL8 后的简单配置
date: 2022-08-09
tags:
  - ubuntu
  - MySQL8
categories: 运维
description: ubuntu 中使用 apt 安装 MySQL8 后关闭 binlog、开放端口、设置root密码和远程登录
---

* apt install mysql-server-8.0

* 设置root密码和远程登录
```shell
sudo mysql -u root # 初始无密码！非root用户必须带上sudo！
use mysql;
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'NewPassword'; # 设置新密码
update user set Host = '%' where User = 'root';
flush privileges;
quit;
```

* 开放端口 & 关闭binlog
```shell
vi /etc/mysql/mysql.conf.d/mysqld.cnf
```
  - 找到 bind-address = 127.0.0.1 这样的一行，将其注释掉
  - 拉到最后面，添加一行内容：skip-log-bin

* 最后重启服务即可生效：
```shell
service mysql restart
```
这样就可以用客户端工具navicat远程连接了。

------
![福利](/images/骚图/三国杀/孙尚香.jpg)

