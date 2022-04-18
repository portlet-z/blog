## 什么是BeanFactory

- 它是ApplicationContext的父接口

- 它才是Spring核心容器，主要的ApplicationContext实现都【组合】了它的功能

## BeanFactory能干点啥

- 表面上只有getBean

- 实际上控制反转，基本的依赖注入，直至Bean的生命周期的各种功能，都由它的实现类提供

## ApplicationContext

- 提供国际化的功能

```java
//新建resources/messages.properties, resources/messages_en.properties, resources/messages_zh.properties三个文件
//messages_en.properties hi=hello
//messages_zh.properties hi=你好
System.out.println(context.getMessage("hi", null, Locale.ENGLISH));
System.out.println(context.getMessage("hi", null, Locale.CHINESE));
```

- 获取资源文件

```java
Resource[] resources = context.getResources("classpath*:META-INF/spring.factories");
//classpath后面加*是获取jar包里面的资源文件
for(Resource resource : resources) {}
```

- 获取环境变量

```java
//获取系统的环境变量，不区分大小
System.out.println(context.getEnvironment().getProperty("java_home"));
//获取配置文件中的变量
System.out.println(context.getEnvironment().getProperty("server.port"));
```

- 事件发布，用于事件解耦

```java
context.publishEvent(new UserRegisterEvent(context));
```

- BeanFactory与ApplicationContext并不仅仅是简单接口继承的关系，ApplicationConte xt组合并扩展了BeanFactory的功能
