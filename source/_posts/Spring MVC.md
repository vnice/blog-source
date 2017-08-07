---
title: Spring MVC
date: 2014-02-15
tags: [Spring]
categories: [Spring 源码分析]
---

# 概要
> Spring MVC 是建立在IoC容器的基础之上的。了解Spring MVC，首先要了解Spring IoC容器是如何在Web环境中被载入，并起作用的

Spring IoC是一个独立的模块，它并不是直接在Web容器中发挥作用的，如果在Web环境中使用IoC容器，需要为IoC设计一个启动过程，将IoC容器载入，并在
Web容器中建立起来，这个过程是和Web容器的启动过程是建立在一起的。在这个过程中，一方面处理Web容器的启动，另一方面通过设定特定的Web容器拦截器或者
监听器，将IoC容器载入到Web环境中来，并将其初始化。在建立完这个过程后，IoC容器才能正常工作。而Spring MVC是建立在IOC容器基础之上的，这样才能建立
其MVC框架的运行机制，从而响应Web容器传递的Http请求。

# Web.xml配置

```xml

    <servlet>
        <servlet-name>mvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/spring-mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
        <async-supported>true</async-supported>
    </servlet>

    <servlet-mapping>
        <servlet-name>mvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

     <context-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/spring-dao.xml,classpath:spring/spring-service.xml</param-value>
     </context-param>


    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

```
在上面的配置中，首先定义了一个Servlet，它是Spring MVC 的DispatcherServlet,在Spring MVC中是一个很重要的类，起着分发请求的作用，同时
这个Servlet对应的url为/，意味着接收所有格式的请求地址。init-param中配置了spring mvc的配置文件，在这个文件中定义了mvc相关的bean,同时定义了
一个listener，这个监听器是与Web服务器生命周期相关联的，由ContextLoaderListener监听器负责完成IoC容器在Web环境中的启动工作。
从上面的配置看到DispatcherServlet，ContextLoaderListener，提供了Web容器中对Spring的接口，也就是说，这些接口与Web容器耦合是通过
ServletContext来实现的，这个ServletContext为SpringIoC容器提供了一个宿主环境，在这个宿主环境中，Spring MVC 建立起了一个IoC容器的体系。
这个IoC容器体系是通过ContextLoaderListeners的初始化建立的。在建立起IoC容器体系后，把DispatcherServlet作为Sping MVC 处理Web请求的转发器
建立起来，从而完成了响应Http请求的准备。

# IoC容器在Web环境中的启动过程

## IoC容器启动的基本过程
IoC容器的启动过程就是建立上下文的过程，该上下文与ServletContext相伴而生的，同时也是IoC容器在Web环境中的具体表现之一。由ContextLoaderListener启动的
上下文为根上下文，在根上下文的基础上，还有一个WebMVC 相关的上下文用来保存控制器MVC需要的对象，作为根上下文的子上下文，构成一个层次化的上下结构体系，这个
上下文体系是由ContextLoader来完成的。

![WebApplicationContext接口的继承关系](/img/inline/webapplicationcontext.png)

在这个类继承关系中 , 可以从熟悉的XmlWebAPPlicationContext入手来了解它的接口实现.在接口设计中 , 最后是通过ApplicationContex接口与BeanFactory接口对接的 ,
而对于具体的功能实现 , 很多都是封装在其基类AbstractRefreshableWebApplicationContext中 完成的。 同样 , 在源代码中 , 也可以分析出类似的继承关系。

在启动过程中，Spring会使用一个默认的WebApplicationContext的实现作为IoC容器就是`XmlApplicationContext`,在继承了基本的ApplicationContext功能的基础
上，增加了Web环境和对Xml配置处理功能，在`XmlApplicationContext`的初始化过程中，Web容器中的IoC容器被建立起来，从而在Web容器中建立起整个Spring应用
这里加载Bean的方式和之前所说的加载`BeanDefinition`一样，只不过多了一些读取指定配置文件的applicationContext.xml文件的细节。

为了了解IoC容器在Web容器中的启动原理 , 这里对启动器ContextLoaderListener的实现 进行分析. 这个监听器是启动根IoC容器并把它载入到Web容器的主要 ,
也是整个 Spring Web应用加载IoC的第一个地方。 从加载过程可以看到, 首先从Servlet事件中得到 ServletContext, 然后可以读取配置在web.xml中的各个相关的属性值,
接着ContextLoader会 实例化WebApplicationContext, 并完成其载入和初始化过程. 这个被初始化的第一个上下文 作为根上下文而存在.
这个根上下文载人后, 被绑定到Web应用程序的ServletContext上。 任何需要访问根上下文的应用程序代码都可以从WebApplicationContextUtils类的静态方法中得到

```java
 public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
        // 先判断WebApplicationContext是不是已经存在，已存在则抛出异常
        if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
            throw new IllegalStateException("Cannot initialize context because there is already a root application context present - check whether you have multiple ContextLoader* definitions in your web.xml!");
        } else {
            Log logger = LogFactory.getLog(ContextLoader.class);
            servletContext.log("Initializing Spring root WebApplicationContext");
            if (logger.isInfoEnabled()) {
                logger.info("Root WebApplicationContext: initialization started");
            }

            long startTime = System.currentTimeMillis();

            try {
                // 没有先创建一个
                if (this.context == null) {
                    this.context = this.createWebApplicationContext(servletContext);
                }

                // 这里判断有没有parent上下文，如果有则把parent上下文加载出来。
                if (this.context instanceof ConfigurableWebApplicationContext) {
                    ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext)this.context;
                    if (!cwac.isActive()) {
                        if (cwac.getParent() == null) {
                            ApplicationContext parent = this.loadParentContext(servletContext);
                            cwac.setParent(parent);
                        }

                        this.configureAndRefreshWebApplicationContext(cwac, servletContext);
                    }
                }

                //将加载完的webapplicationcontext设置到servletContext中保存起来。
                servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);

               //... 省略部分代码
        }
    }
```
创建WebApplicationContext的代码

```java
 protected WebApplicationContext createWebApplicationContext(ServletContext sc) {
        // 这里判断用什么样的类作为Web 容器中Ioc容器
        Class<?> contextClass = this.determineContextClass(sc);
        if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
            throw new ApplicationContextException("Custom context class [" + contextClass.getName() + "] is not of type [" + ConfigurableWebApplicationContext.class.getName() + "]");
        } else {
            // 直接实例化要产生的IoC容器
            return (ConfigurableWebApplicationContext)BeanUtils.instantiateClass(contextClass);
        }
    }

    protected Class<?> determineContextClass(ServletContext servletContext) {
        String contextClassName = servletContext.getInitParameter("contextClass");
        if (contextClassName != null) {
            try {
                return ClassUtils.forName(contextClassName, ClassUtils.getDefaultClassLoader());
            } catch (ClassNotFoundException var4) {
                throw new ApplicationContextException("Failed to load custom context class [" + contextClassName + "]", var4);
            }
        } else {
            contextClassName = defaultStrategies.getProperty(WebApplicationContext.class.getName());

            try {
                return ClassUtils.forName(contextClassName, ContextLoader.class.getClassLoader());
            } catch (ClassNotFoundException var5) {
                throw new ApplicationContextException("Failed to load default context class [" + contextClassName + "]", var5);
            }
        }
    }
```
