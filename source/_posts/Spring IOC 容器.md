---
title: Spring IOC 容器
date: 2014-02-04
tags: [Spring]
---

# IOC容器的设计与实现
Spring IOC 容器中有两个主要的容器

 - 实现`BeanFactory` 接口的简单容器实现，只是实现了容器的基本功能，主要的几个方法如下
    - 其中最主要的是`getBean`方法，通过指定的名字对在容器中获取Bean
    - `containsBean`判断容器中是否包含指定名字的bean
    - `isSingletion`和`isPrototype`方法分别判断bean是否是单例或者多例的
    - `isTypeMatch` 来查询指定名字的Bean 的Class类型是否和特定的Class类型匹配
 - 实现`ApplicationContext`接口的高级容器实现
    - 扩展了BeanFactory的主要功能，同时支持国际化实现，访问资源的功能，支持应用事件。

# BeanFactory和FactoryBean
   `BeanFactory` 是一个Factory，也就是IoC容器中的对象工厂
    `FactoryBean` 是一个Bean,在Spring中所有的Bean都是由`BeanFactory`来管理的，但是对`FactoryBean`而言，这个Bean不是简单的Bean,而是一个能产生或者修饰对象生成的工厂Bean

# BeanDefinition
 在Spring 提供的基本IoC容器的接口定义和实现的基础上，Spring通过定义`BeanDefinition`来管理基于Spring的应用中各种对象以及他们之间相互依赖关系，`BeanDefinition`
 抽象了我们队Bean的定义，对IoC容器来说，`BeanDefinition`就是对依赖反转模式中管理的对象依赖关系中的数据抽象，也是容器实现依赖反转功能的核心数据结构。

# IoC容器的初始化
各种不同的容器初始化都是在加载完配置文件后调用`refresh`方法，这个方法标志这IoC容器的正式启动，这个启动包括`BeanDefinition`的Resource定位，载入和注册三个基本过程

## Resource 定位
  通过ResourceLoader 对不同形式的Resource进行加载，比如`FileSystemResource`,`ClassPathResource`
## Beandefinition 载入
  将用户定义好的Bean表示为IoC容器的内部数据结构就是`BeanDefinition`，通过这个对象，IoC容器能够方便的对Bean进行管理
## IoC容器注册BeanDefinition
  通过调用BeanDefinitionRegistry的实现，将载入过程中得到的BeanDefinition向IoC容器中注册，实际上是是添加到一个HashMap中去，IoC容器就是通过这个HashMap来持有这些BeanDefinition数据的
  >   这里的IoC容器初始化过程们，一般不包含Bean的依赖注入的实现。在Spring IoC的设计过程中，Bean 定义的载入和依赖注入是两个独立的过程
  依赖注入一般发生在应用第一次通过getBean向容器索取Bean的时候。

# IoC容器的依赖注入
 ![依赖注入时序图](/img/spring_ioc_process.png)


