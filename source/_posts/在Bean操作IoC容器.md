---
title: 在Bean中操作IoC容器
date: 2014-02-13
tags: [Spring]
---

IoC容器管理的Bean一般不需要了解容器的状态和直接使用容器，为了和IoC容器api耦合降到最低。但是在某些情况下，是需要在Bean
里面直接对IoC容器进行操作，这个时候，需要在Bean中设定对容器的感知。Spring IoC容器提供了该功能，它是通过特定的aware
接口来完成的主要有以下这些:

* `BeanNameAware` 可以在Bean中得到它所在的IoC容器中的Bean的示例名称。
* `BeanFactoryAware` 可以在Bean中得到它所在IoC容器，从而直接在Bean中使用IoC容器的服务。
* `ApplicationContextAware` 可以在Bean中得到Bean所在应用上下文，从而直接在Bean中使用应用上下文服务。
* `MessageSourceAware` 在Bean中可以得到消息源
* `ApplicationEventPublisherAware` 在Bean中可以得到应用上下文的事件发布器，从而可以在Bean中发布应用上下文事件
* `ResourceLoaderAware` 在Bean中可以得到ResourceLoader,从而在Bean中使用ResourceLoader加载外部对应的Resource资源
