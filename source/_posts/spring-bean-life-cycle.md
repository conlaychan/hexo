---
title: SpringBean的生命周期
date: 2022-01-10
tags:
  - spring
categories: java
description: SpringBean的生命周期
---

### 简而言之

简单来说就是：<font color="green">构造器 -> 依赖注入 -> 各种各样的 aware -> 自定义初始化 -> 自定义销毁</font>，InstantiationAwareBeanPostProcessor 穿插在<font color="green">构造器</font>前后，BeanPostProcessor 穿插在<font color="green">自定义初始化</font>前后。

### 详细顺序

* <font color="red">InstantiationAwareBeanPostProcessor</font>#postProcess<font color="red">Before</font>Instantiation
* 构造器
* <font color="red">InstantiationAwareBeanPostProcessor</font>#postProcess<font color="red">After</font>Instantiation
* <font color="red">InstantiationAwareBeanPostProcessor</font>#postProcessPropertyValues
* 依赖注入
* 各种各样的 aware（如：BeanNameAware、ApplicationContextAware）
* <font color="blue">BeanPostProcessor</font>#postProcess<font color="blue">Before</font>Initialization
* @PostConstruct
* InitializingBean#afterPropertiesSet
* <font color="blue">BeanPostProcessor</font>#postProcess<font color="blue">After</font>Initialization
* @PreDestroy
* DisposableBean#destroy

注意：一个 <font color="red">InstantiationAwareBeanPostProcessor</font> 或 <font color="blue">BeanPostProcessor</font> 是对所有 spring bean 都生效的！

------
![福利](/images/骚图/三国杀/何太后.jpg)
