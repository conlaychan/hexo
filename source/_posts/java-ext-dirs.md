---
title: 使用 java.ext.dirs 加载额外的 lib 目录
date: 2022-01-10
tags:
  - jvm
categories: java
description: 使用 java.ext.dirs 加载额外的 lib 目录
---

* 众所周知：执行 java 命令时指定参数 java.ext.dirs 可令 jvm 额外加载特定目录下的 jar 文件
* java.ext.dirs 本身是有默认值的，所以指定该参数时通常建议把默认值带上，其默认值为：
    * $JAVA_HOME/jre/lib/ext （Linux）
    * %JAVA_HOME%/jre/lib/ext （Windows）
* 假设我们要令 jvm 额外加载相对路径 lib 下的 jar 文件，完整命令为：
    * java -Djava.ext.dirs="$JAVA_HOME/jre/lib/ext<font color="red">**:**</font>lib" -jar hello.jar （Linux，用冒号分隔，引号可有可无）
    * java -Djava.ext.dirs="%JAVA_HOME%/jre/lib/ext<font color="red">**;**</font>lib" -jar hello.jar （Windows，用分号分隔，必须有引号）
* 注意：以上我都是假定你正确配置了环境变量 JAVA_HOME 指向 JDK 的主目录！
* 测试环境：JDK8

------
![福利](/images/骚图/三国杀/双乔.jpg)

