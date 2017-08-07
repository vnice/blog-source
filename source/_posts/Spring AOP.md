---
title: Spring AOP
date: 2014-02-15
tags: [Spring]
categories: [Spring 源码分析]
---

> AOP 是Spring框架的核心功能模块之一。AOP与IoC容器的结合使用，为应用开发
或者Spring自身功能的扩展都提供了便利。Spring AOP的实现和其他特性的实现一样，除了
可以使用Spring 本身提供的AOP实现之外，还封装了业界优秀的AOP解决方案AspectJ来供使用。

# Advice 通知

Advice通知定义在连接点做什么，为切面增强提供织如接口。在Spring AOP 中它主要描述Spring AOP围绕方法
调用而注入的切面行为，在SpringAOP 实现使用了AOP联盟定义的同一接口org.aopalliance.aop.Advice，
通过这个接口，为AOP切面增强的织入功能做了更多的细化和扩展，比如提供了更具体的通知类型，如BeforeAdvice,
AfterAdvice,ThrowsAdvice.作为Spring AOP 定义的接口类，具体的切面增强可以通过这些接口集成到AOP框架中去发挥作用。

## BeforeAdvice
在`BeforeAdvice` 的继承关系关系中，定义了为待增强的目标方法设置 前置增强接口`MethodBeforeAdvice`

## AfterReturningAdvice
在目标方法调用结束并成功返回的时候，会执行回调函数`afterReturning`

## ThrowsAdvice
在抛出异常时会回调`afterThrowing`方法

# Pointcut 切点
切点决定Advice通知应该作用于哪个连接点，也就是说通过Pointcut来定义需要增强的方法的集合
切点的类继承关系如下

![pointcut](/img/inline/pointcut.png)

# Advisor
完成对目标方法的切面增强设计(Advice)和关注点的设计(Pointcut)以后，需要一个对象将他们结合起来，完成这个
作用的就是Advisor通知器，把Advice和Pointcut结合起来，这个结合为使用IoC容器配置AOP应用，或者说即开即用的AOP基础设施，提供了便利


# JVM 动态代理的特性
在Spring AOP中，使用核心技术的是动态代理，而这种动态代理实际上是JDK的一个特性。通过JDK的动态代理，可以为任意
对象创建代理对象，对弈具体使用来说，这个特性是通过Java Reflection API 来完成的。

# 设计原理
在Spring的AOP模块中，一个主要的部分是代理对象的生成，而对于Spring应用，可以看到，是通过配置和调用Spring的ProxyFactoryBean来完成这个任务的
在ProxyFactoryBean中封装了主要的代理对象的生成过程。在这个过程中可以使用JDK的Proxy或者是CGLIB

在分析Spring AOP 的实现原理中，主要以ProxyFactoryBean的实现作为例子和实现的基本线索进行分析
这是因为ProxyFactoryBean是在Spring IoC环境中创建 AOP 应用的底层方法，也是最灵活的方法，Spring通过它
完成了对AOP使用封装。以ProxyFactoryBean实现为入口，逐层深入是学习理解Spring AOP 实现的学习路径

![AopProxy生成过程](/img/inline/proxyfactorybean.png)