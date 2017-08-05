---
title: Spring 基本概念
date: 2014-02-01
tags: [Spring]
---
# 用途

使用Spring框架来简化开发，实现程序的解耦，以及使用Spring一些开箱既用的功能，以及对常用框架的一些分装

其中解耦体现在对象的解耦和通用功能的解耦分别是IOC和AOP


# IOC 控制反转
> 在不使用IOC容器的情况下，开发者需要自己new 对象，当业务变大之后，new 的对象也就变得非常多，对象之间的依赖关系也就变得很复杂，耦合度过高不好维护，不仅出现在对象和对象之间，也出现在模块与模块之间，各个系统之间。

> IOC 就是用来解决对象之间的耦合问题，将原本开发者自己需要new 对象的事情交给IOC第三方 容器来实现，实现对象之间的解耦，模块之间的解耦，系统之间的解耦

> 比如UserController 这个控制器，依赖UserService,UserService 又依赖UserDAO,UserDAO 依赖一些ORM框架。这里不使用IOC的情况下你直接在UserController中new UserService()是不行的，需要从最底层开始，先初始化ORM框架，然后new DAO层，将ORM框架set到DAO层，然后new Service 层，将DAO层set到Service层，最后将service层set到Controller层。这样开发者需要关心各个层之间的依赖，在使用了IOC容器之后，你在Controller层不用关心Service层里依赖了什么对象。直接使用Service对象，如果service层中的对象不存在，则有IOC容器初始化service中依赖，等等。

> 将这个整个过程反转过来 就是控制反转，整个依赖对象对象的set过程就是依赖注入。注入的对象都被当做一个普通的Bean存在IOC容器当中，

> 当然上面依赖注入只是控制反转的一种形式，也可以通过观察者模式，模版方法模式实现反转


# AOP 面向切面
> 将影响多个类的的行为分装到可重用的模块中，将一些公共功能抽离出来，开发者只关注业务相关的内容。，分别开发，在运行时，将切面分装的公共功能横切到业务层中。达到所需要的功能。常用的如事务管理、安全检查、缓存、对象池管理等

## AOP的关键在AOP框架创建的代理

- 代理主要分静态代理和动态代理
    - 静态代理是编译时期通过AOP框架的命令生成AOP代理类，AspectJ 编译时期的增强，
    - 动态代理是在运行时期借助JDK的Proxy或者CGLIB在内存中临时生成AOP的代理类，运行时期的增强
    - 如何选择JDK Proxy 和CGLIB， 如果目标类没有实现接口，则使用CGLIB，如果实现了接口则使用JDKProxy

- JDK Proxy
根据类加载器和接口创建代理类，生成一个目标对象的接口的实现类(InvocationHandler)

- CGLIB 一个代码生成类库，在运行期生成目标对象的一个子类，子类对父类的各种方法增强(MethodInterceptor)


