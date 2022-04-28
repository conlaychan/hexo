---
title: Ubuntu常用命令
date: 2022-01-11 17:07:35
tags:
  - ubuntu
  - linux
categories: 运维
description: linux（Ubuntu）常用命令：远程密码登录、软件源、磁盘查看、docker、apt-daily、端口查看
---

### 查看Linux版本

```shell
lsb_release -a
```

### 启用 root 账号

#### 1、为root用户设置密码

```shell
sudo passwd root
```

#### 2、修改ssh的配置文件，开启远程登录

> 编辑文件 /etc/ssh/sshd_config，找到  PermitRootLogin ，将其值设为 yes

### 修改Ubuntu软件源

建议安装系统阶段就指定mirror地址为  http://mirrors.163.com/ubuntu/  。

若已安装完系统，Ubuntu 的软件源配置文件是 /etc/apt/sources.list。将系统自带的该文件做个备份，写入新内容：

> deb http://mirrors.163.com/ubuntu/ focal main restricted universe multiverse
> 
> deb http://mirrors.163.com/ubuntu/ focal-security main restricted universe multiverse
> 
> deb http://mirrors.163.com/ubuntu/ focal-updates main restricted universe multiverse
> 
> deb http://mirrors.163.com/ubuntu/ focal-proposed main restricted universe multiverse
> 
> deb http://mirrors.163.com/ubuntu/ focal-backports main restricted universe multiverse

然后执行 apt-get update 更新下列表。

### 设置命令行前面的颜色

> 在 ~/.bashrc 中添加以下内容：

```bash
export PS1="\[\033[01;32m\]\u@\h\[\033[00m\]$ "
```

> 然后执行  source ~/.bashrc

### 列出已安装的软件包
```shell
apt list --installed
```

### 查看磁盘容量和使用量

```shell
df -hl
```

### 查看当前目录下各目录的大小

```shell
du -sh *
```

### docker

#### 1、安装

```shell
apt install docker.io
```

#### 2、启动 docker 服务

```shell
sudo service docker start
```

#### 3、为普通用户设置docker权限

```shell
sudo groupadd docker
sudo usermod -aG docker $USER
然后重新登录
```

#### 4、stop 所有容器

```shell
docker stop $(docker ps -a -q)
```

#### 5、删除所有已停掉的容器

```shell
docker rm $(docker ps -a -q)
```

#### 6、设置 docker 镜像服务器

> 在文件 /etc/docker/daemon.json 中添加以下内容
```json
 {
    "registry-mirrors" : ["https://docker.mirrors.ustc.edu.cn"]
 }
```
>
> 然后重启docker服务
>
> service docker restart

#### 7、连接到其他机器上的 docker 服务

```shell
export DOCKER_HOST=tcp://127.0.0.1:2375 （仅对本次会话有效，非永久）
```

### rpm 安装包转 deb 安装包

```shell
sudo alien -d --script  jdk-8u212-linux-x64.rpm
```

### 安装 deb 包

```shell
sudo dpkg -i jdk1.8_1.8.0212-1_amd64.deb
```

### nohup 命令禁止产生输出文件

```shell
nohup java -jar app.jar >/dev/null 2>&1 &
```

### 关闭 apt-daily.service

```shell
apt-get remove unattended-upgrades
systemctl kill --kill-who=all apt-daily.service
systemctl stop apt-daily.timer
systemctl disable apt-daily.timer
systemctl stop apt-daily.service
systemctl disable apt-daily.service
systemctl mask apt-daily.service
systemctl daemon-reload
```

关闭完成后检查结果：

```shell
systemctl status apt-daily.timer
```

注意结果中的：

> Active: inactive (dead)

### shell脚本中获取脚本文件自身所在目录

```shell
basepath=$(cd `dirname $0`; pwd)
```

### LINUX 中查看端口被占用

```shell
netstat -anp | grep 端口号
```

看监控状态为 LISTEN 表示已经被占用（LISTENING 并不表示端口被占用），如 22 端口：

> tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1005/sshd

1005 是 pid，sshd 是进程名称。

以下命令可列出全部端口：

```shell
netstat -nultp
```

### Windows 中查看端口被占用

```shell
netstat -ano|findstr 10000
```

看监控状态为 LISTENING 表示已经被占用：

> TCP    0.0.0.0:10000          0.0.0.0:0              LISTENING       8884

8884 是 pid。

以下命令可列出全部端口：

```shell
netstat -ano
```

------
![福利](/images/骚图/三国杀/大乔.jpg)
