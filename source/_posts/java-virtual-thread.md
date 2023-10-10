---
title: java21虚拟线程与springboot3集成环境配置
date: 2023-09-25 10:30:00
tags:
  - java21
  - jdk21
  - 虚拟线程
  - spring
  - tomcat
  - hikari
  - lettuce
categories: java
description: java21虚拟线程与spring环境集成配置tomcat、hikari、lettuce、scheduler
---

# 配置虚拟线程的载体线程池

如果服务器CPU核心数太多，且程序并不需要支持太多的并发量，则强烈建议配置一下，以减少使用的平台线程数。jdk源码位置：<font color="blue">java.lang.VirtualThread#createDefaultScheduler</font>，开放了3个配置项，对这3个选项的修改必须在加载<font color="blue">java.lang.VirtualThread</font>类之前！

下面通过spring集成环境来配置虚拟线程的载体线程池，但如果有特殊代码执行在<font color="blue">ApplicationEnvironmentPreparedEvent</font>事件之前且这些特殊代码导致了<font color="blue">java.lang.VirtualThread</font>类在此spring事件之前被加载，那么强烈建议修改优化这些特殊代码的执行时机，否则就只能把配置时机提前到main方法里，那就失去了spring集成能力。

```java
@SpringBootApplication
public class DemoApplication implements ApplicationListener<ApplicationEnvironmentPreparedEvent> {
    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(DemoApplication.class);
        application.addListeners(new DemoApplication());
        application.run(args);
    }
    @Override
    public void onApplicationEvent(ApplicationEnvironmentPreparedEvent event) {
        ConfigurableEnvironment environment = event.getEnvironment();
        String parallelism = environment.getProperty("jdk.virtual-thread-scheduler.parallelism");
        String maxPoolSize = environment.getProperty("jdk.virtual-thread-scheduler.max-pool-size");
        String minRunnable = environment.getProperty("jdk.virtual-thread-scheduler.min-runnable");
        if (parallelism != null) {
            // jdk中的默认值为CPU的逻辑核心数，如果服务器CPU核心数太多应当考虑设置下！
            System.setProperty("jdk.virtualThreadScheduler.parallelism", parallelism);
        }
        if (maxPoolSize != null) {
            System.setProperty("jdk.virtualThreadScheduler.maxPoolSize", maxPoolSize);
        }
        if (minRunnable != null) {
            System.setProperty("jdk.virtualThreadScheduler.minRunnable", minRunnable);
        }
    }
}
```
application.yml 示例：
```yaml
jdk:
  virtual-thread-scheduler:
    parallelism: 4
    max-pool-size: 16
    min-runnable: 1
```

# 配置Java11新增的HttpClient

```java
public class DemoApplication {
    public static final HttpClient HTTP_CLIENT = HttpClient.newBuilder()
            .cookieHandler(new CookieManager())
            .connectTimeout(Duration.ofMillis(100))
            .executor(Executors.newCachedThreadPool(Thread.ofVirtual().name("HttpClient-v", 1).factory()))
            .build();
}
```

# tomcat

```java
@SpringBootApplication
public class DemoApplication {
    @Bean
    public TomcatConnectorCustomizer tomcatConnectorCustomizer(ServerProperties serverProperties) {
        ServerProperties.Tomcat.Threads threads = serverProperties.getTomcat().getThreads();
        ThreadFactory factory = Thread.ofVirtual().name("tomcat-v", 1).factory();
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                threads.getMinSpare(), threads.getMax(), 60L, TimeUnit.SECONDS,
                new SynchronousQueue<>(), factory, new ThreadPoolExecutor.AbortPolicy()
        );
        return connector -> connector.getProtocolHandler().setExecutor(executor);
    }
}
```

application.yml 示例：

```yaml
server:
  tomcat:
    threads:
      max: 21_0000_0000 # 放心大胆得使用虚拟线程（默认200）
      min-spare: 0      # 空载时销毁所有虚拟线程（也可以保持默认10）
```

# lettuce（redis客户端）

```java
@SpringBootApplication
public class DemoApplication {
    @Bean
    public ClientResourcesBuilderCustomizer lettuceClientResourcesCustomizer() {
        ThreadFactoryProvider threadFactoryProvider = poolName -> Thread.ofVirtual().name(poolName + "-v", 1).factory();
        return customizer -> customizer.threadFactoryProvider(threadFactoryProvider);
    }
}
```

# HikariDataSource（数据库连接池）

```java
@SpringBootApplication
public class DemoApplication implements InstantiationAwareBeanPostProcessor{
    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        if (bean instanceof HikariDataSource hikari) {
            hikari.setThreadFactory(Thread.ofVirtual().name("hikari-v", 1).factory());
        }
        return true;
    }
}
```

# spring 定时任务

不要通过增加<font color="blue">spring.task.scheduling.pool.size</font>的值来达成定时任务的并发执行，因为此数量之内的线程不会被回收，而应通过使用<font color="blue">@EnableAsync</font>和<font color="blue">@Async</font>异步机制来实现。

不要将任何<font color="blue">java.util.concurrent.Executor</font>的类注册为spring bean，否则会导致spring自带的默认<font color="blue">ThreadPoolTaskExecutor</font>失效，除非将此<font color="blue">Executor</font>类型的spring bean的名称设为<font color="blue">applicationTaskExecutor</font>和<font color="blue">taskExecutor</font>，但这样做又会带来更多issue。

详见spring源码位置：<font color="blue">TaskExecutionAutoConfiguration#applicationTaskExecutor</font>。

```java
@EnableAsync
@EnableScheduling
@SpringBootApplication
public class DemoApplication{
    @Bean
    public TaskSchedulerCustomizer taskSchedulerCustomizer(TaskSchedulingProperties schedulingProperties) {
        String threadNamePrefix = schedulingProperties.getThreadNamePrefix() + "v";
        ThreadFactory factory = Thread.ofVirtual().name(threadNamePrefix, 1).factory();
        return customizer -> customizer.setThreadFactory(factory);
    }

    @Bean
    public TaskExecutorCustomizer taskExecutorCustomizer(TaskExecutionProperties executionProperties) {
        String threadNamePrefix = executionProperties.getThreadNamePrefix() + "v";
        ThreadFactory factory = Thread.ofVirtual().name(threadNamePrefix, 1).factory();
        return customizer -> customizer.setThreadFactory(factory);
    }
}
```

application.yml 示例：

```yaml
spring:
  task:
    scheduling:
      pool:
        size: 1 # 保持默认值，使用一个（虚拟）线程进行调度就够了，至于并发问题则用异步机制来解决
    execution:
      pool:
        core-size: 0         # 无任务时销毁所有虚拟线程（也可以保持默认8）
        max-size: 2147483647 # 保持默认值：Integer.MAX_VALUE，放心大胆得使用虚拟线程
        queue-capacity: 0    # 确保并发执行定时任务的最重要设置
```


------
![福利](/images/骚图/三国杀/孙鲁班.jpg)

