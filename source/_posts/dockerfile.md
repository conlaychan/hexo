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
```text
FROM openjdk:8-jre

RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" >> /etc/timezone
ENV TZ Asia/Shanghai
```
### shell
```shell
docker build -t openjdk8:local .
docker rmi openjdk:8-jre
docker tag openjdk8:local openjdk:8-jre
```

## openjdk:21-jre 东八区（jdk17同样做法）
### Dockerfile
```text
FROM eclipse-temurin:21-jre

RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" >> /etc/timezone
ENV TZ Asia/Shanghai
```
### shell
```shell
docker build -t openjdk:21-jre .
docker rmi eclipse-temurin:21-jre
```

## ubuntu:20.04 东八区
### Dockerfile
```text
FROM ubuntu:20.04

ENV DEBIAN_FRONTEND noninteractive #tzdata 静默认安装

RUN apt update \
  && apt install -y tzdata --no-install-recommends \
  && echo "Asia/Shanghai" > /etc/timezone  \
  && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
  && dpkg-reconfigure -f noninteractive tzdata \
  && apt clean \
  && apt autoremove -y \
  && rm -rf /var/lib/apt/lists/*
```
### shell
```shell
docker build -t ubuntu20:local .
docker rmi ubuntu:20.04
docker tag ubuntu20:local ubuntu:20.04
```

## ubuntu:20.04-fat
含有openjdk8以及各种网络工具并修改了时区，可用于测试docker运行环境。
### Dockerfile
```text
FROM ubuntu:20.04

ENV DEBIAN_FRONTEND noninteractive #tzdata 静默认安装

RUN apt update \
  && apt install -y tzdata telnet wget net-tools iputils-ping openjdk-8-jre-headless --no-install-recommends \
  && echo "Asia/Shanghai" > /etc/timezone  \
  && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
  && dpkg-reconfigure -f noninteractive tzdata \
  && apt clean \
  && apt autoremove -y \
  && rm -rf /var/lib/apt/lists/*
```
### shell
```shell
docker build -t ubuntu:20.04-fat .
```

## centos:7-fat
含有openjdk8以及各种网络工具并修改了时区，可用于测试docker运行环境。
### Dockerfile
```text
FROM centos:7

RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" >> /etc/timezone \
    && localedef -c -f UTF-8 -i zh_CN zh_CN.utf8 \
    && yum -y install java-1.8.0-openjdk net-tools telnet wget \
    && yum clean all \
    && rm -rf /var/cache/yum/*

ENV TZ Asia/Shanghai
ENV LANG zh_CN.utf8
ENV LC_ALL zh_CN.utf8
```
### shell
```shell
docker build -t centos:7-fat .
docker rmi centos:7
```

------
![福利](/images/骚图/三国杀/孙尚香2.jpg)

