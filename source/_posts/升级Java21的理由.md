---
title: 升级Java21的理由
date: 2024-01-10
tags:
  - 虚拟线程
  - spring
  - GraalVM
categories: java
description: 升级Java21的理由，排名分先后（按照每个特性的吸引力从高到低）
---


# Java各版本支持周期

[官方链接](https://www.oracle.com/java/technologies/java-se-support-roadmap.html)

下面只列出LTS版本：

| Release | GA Date   | Premier Support Until | Extended Support Until |
| ------- | --------- | --------------------- | ---------------------- |
| 8       | 2014年3月 | 2022年3月             | 2030年12月             |
| 11      | 2018年9月 | 2023年9月             | 2032年1月              |
| 17      | 2021年9月 | 2026年9月             | 2029年9月              |
| 21      | 2023年9月 | 2028年9月             | 2031年9月              |
| 25      | 2025年9月 | 2030年9月             | 2033年9月              |

以下排名分先后（按照每个特性的吸引力从高到低）

# 虚拟线程（since 21）

最炸裂的新功能，如果这都不能让你下定决心升级Java21，那么剩下的内容你都不需要看了。

虚拟线程可在不增加平台线程的情况下大幅提升BIO系统的吞吐量，典型代表：

- tomcat等servlet容器
- 数据库连接池
- redis连接池
- 使用 Java 请求一个 HTTP 地址
- 使用了 Thread#sleep 方法的场景中

但在某些场景中虚拟线程没有帮助：

- netty，NIO框架，其线程模型已经足够优秀
- 计算密集型系统，即主要消耗CPU计算能力的场景
- 使用了 synchronized 锁的场景中，虚拟线程在等待 synchronized 锁时不会释放其载体线程

# spring boot 3（since 17）

spring boot 3 要求 JDK 版本至少为17。

如果你还想跟上开源社区的节奏，直接一步到位升级到 Java 21 吧。

# GraalVM 原生编译（since 17）

GraalVM 可将 Java 应用编译为二进制可执行程序，运行时不再依赖 JVM，spring 开发团队做了大量工作使基于 spring boot 的 Java 应用可更轻松地完成原生编译。

# JDK 内置 HttpClient（since 11）

JDK 内置的 HttpClient 使开发者用 Java 代码请求一个 http 地址的姿势变得标准化，开发者终于可以抛弃笨重的 apache http component 以及其他各种千奇百怪的 http 库啦！

此 HttpClient 支持同步和异步两套 API，其中异步 API 依赖一个线程池来实现回调，且可以自定义为虚拟线程池。

此 HttpClient 支持自动处理典型的cookie机制：收到服务端响应的cookie后会在后续的请求中自动携带所有cookie。

# switch-case表达式（since 21）

switch-case 以前是语句，现在是表达式了，因为它有了返回值。

以前用冒号和break关键字来写一个case，现在不需要写反直觉的break了，改为写一个箭头。

以前一个case只能匹配一个值，现在可以匹配多个了。

switch 判断的对象可以是 null 了。

```java
return  switch (day) {
    case "Monday","Tuesday","Wednesday","Thursday","Friday" -> "工作日";
    case "Saturday", "Sunday" -> "休息日";
    case null -> "";
    default->"未知";
};
```

模式匹配：switch 现在可以判断目标对象的类型了：

```java
return switch (obj) {
    case Integer i -> "It is an integer";
    case String s -> "It is a string";
    default -> "It is none of the known data types";
};
```

------
![福利](/images/骚图/三国杀/孙鲁班2.jpg)
