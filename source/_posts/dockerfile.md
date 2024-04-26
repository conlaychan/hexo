---
title: 常用docker镜像的构建文件
date: 2023-03-17
tags:
  - docker
categories: 运维
description: openjdk8、ubuntu20修改时区，centos7内置测试工具
---

## openjdk:8-jre 东八区
### Dockerfile

vim Dockerfile_openjdk8

```text
FROM openjdk:8-jre

RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" >> /etc/timezone
ENV TZ Asia/Shanghai
```
### shell
```shell
docker build -t openjdk8:local -f Dockerfile_openjdk8 .
docker rmi openjdk:8-jre
docker tag openjdk8:local openjdk:8-jre
docker rmi openjdk8:local
```

## openjdk:21-jre 东八区（jdk17同样做法）
### Dockerfile

vim Dockerfile_openjdk21

```text
FROM eclipse-temurin:21-jre

RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" >> /etc/timezone
ENV TZ Asia/Shanghai
```
### shell
```shell
docker build -t openjdk:21-jre -f Dockerfile_openjdk21 .
docker rmi eclipse-temurin:21-jre
```

## ubuntu:20.04
含有各种网络工具并修改了时区，可用于测试docker运行环境。
### Dockerfile

vim Dockerfile_ubuntu20

```text
FROM ubuntu:20.04

ENV DEBIAN_FRONTEND noninteractive #tzdata 静默安装

RUN apt update \
  && apt install -y tzdata telnet wget net-tools iputils-ping --no-install-recommends \
  && echo "Asia/Shanghai" > /etc/timezone  \
  && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
  && dpkg-reconfigure -f noninteractive tzdata \
  && apt clean \
  && apt autoremove -y \
  && rm -rf /var/lib/apt/lists/*
```
### shell
```shell
docker build -t ubuntu20:local -f Dockerfile_ubuntu20 .
docker rmi ubuntu:20.04
docker tag ubuntu20:local ubuntu:20.04
docker rmi ubuntu20:local
```

## CentOS:7
含有各种网络工具并修改了时区，可用于测试docker运行环境。
### Dockerfile

vim Dockerfile_centos7

```text
FROM centos:7

RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" >> /etc/timezone \
    && localedef -c -f UTF-8 -i zh_CN zh_CN.utf8 \
    && yum -y install net-tools telnet wget \
    && yum clean all \
    && rm -rf /var/cache/yum/*

ENV TZ Asia/Shanghai
ENV LANG zh_CN.utf8
ENV LC_ALL zh_CN.utf8
```
### shell
```shell
docker build -t centos7:local -f Dockerfile_centos7 .
docker rmi centos:7
docker tag centos7:local centos:7
docker rmi centos7:local
```

## MySQL:8 东八区

### Dockerfile

vim Dockerfile_mysql8

```shell
FROM mysql:8.0.22

RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" >> /etc/timezone
ENV TZ Asia/Shanghai
```

### shell

```shell
docker build -t mysql8:local -f Dockerfile_mysql8 .
docker rmi mysql:8.0.22
docker tag mysql8:local mysql:8.0.22
docker rmi mysql8:local
```



------
![福利](/images/骚图/三国杀/孙尚香2.jpg)

