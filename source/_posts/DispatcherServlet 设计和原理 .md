---
title: DispatcherServlet 设计和原理
date: 2014-02-18
tags: [Spring]
categories: [Spring 源码分析]
---
在Spring MVC中 ,看到了在web.xml中 , 除了需要配置ContextLoaderListener之外 , 还要对 DispatcherServlet迸行配置作为一个Servlet,
这个DispatcherServlet实现的是Sun的J2EE核 心模式中的前端控制器模式 (Front Control1er), 作为一个前端控制器 ,
所有的Web诘求都需要通过它来处理 , 进行转发 、 匹配、 数据处理后 , 井转由页面迸行展现 , 因此这个 DispatcherServlet可以看成是Spring MVC实现中最为核心的部分, 它的设计与分析也是下面分 祈Spring MVC的一条主线。

在完成对ContextLoaderListener的初始化以后 , Web容器开始初始化DispatcherServlet, 这个初始化的启动与在web.xml中对载入次序的定义有关。 DispatcherServlet会建立 自 己的上 下文来持有Spring MVC的Bean对象 ,  在建立这个自己持有的loC容器时,
会从ServletContext中得到根上下文作为DispatcherServlet持有上下文的双亲上下文，有了这个根上下文，再对自己持有的根上下文进行初始化，最后把自己持有的这个上下文保存到ServletContext，供以后使用。

为了解这个过程 , 可以从DispatcherServlet的父类 FrameworkServlet的代码入手, 去探寻DispatcherServlet的启动过程, 它同时也是Spring MVC的启动过程. ApplicationContext的创建过程和ContextLoader创建根上下文的过程有许多类似的地方.

从源码的分析中发现DispatcherServlet的类继承关系为`HttpServletBean`->`FrameworkServlet`->`DispatchServlet`

具体执行过程如下

![DispatcherServlet执行过程](/img/inline/dispatcherservlet.png)

作为一个Servlet ，Web容器会调用Servlet的doGet或者doPost方法，在经过FrameworkServlet的processRequest的简单处理后，会调用DispatcherServlet的doService方法，
在这个方法调用中，封装课doDispatch(),这个doDispatch是Dispatch实现MVC模式的主要部分。

# DispatchServlet的启动和初始化
作为Servlet, DispatcherServlet的启动与Servlet的启动过程是相联系的。在Servlet的初始化过程中, Servlet的init方法会被调用, 以迸行初始化， DispatcherServlet的基类 HtlpServletBean中的这个初始化过程如上图所示。 在初始化开始时, 需要读取配置在ServletContext中 的Bean属性参数, 这些属性参数设置在web.xml的web容器初始化参数中 . 使用编程式的方式来设置这些Bean属性, 在这里可以看到对PropertyValues和BeanWrapper的使用. 对于这些和依赖注入相关的类的使用, 在分析IOC容器的初始化时. 尤其是在依赖注入实现分析时, 有提到过。 只是这里的依赖注入是与web容器初始化相关的, 初始化过程由HttpServletBean来 完成。 接着会执行DispatchServlet持有的IOC容器的初始化过程, 在这个初始化过程中, 一个新的上下文被建立起来, 这个DispatchServlet持有的上下文被设置为根上下文的子上下文. 可以认为, 根上下文是和Web应用相对应的一个上下文, 而DispatchServlet持有的上下文是 和Servlet对应的一个上下文. 在一个Web应用中, 往往可以容纳多个Servlet存在 与此相对应，对于应用在Web容器中的上下文体系，一个根上下文可以作为许多Servlet上下文的双亲上下文。
了解IoC工作原理都知道在向IoC容器中getBean时，IoC容器会首先向其双亲上下文去getBean，也就是说，在根上下文中定义的Bean是可以被各个Servlet持有的上下文得到和共享的。
DispatcherServlet上下文在建立起来以后，也需要和其他IoC容器一样完成初始化，这个初始化也是通过resfresh方法完成的。最后DispatchServlet给这个
自己持有的上下文命名，并把它设置到Web容器的上下文中，这个名称和在Web.xml设置的DispatchServlet的Servlet名称有关。从而保证了这个上下文在Web环境上下文体系中的唯一性

`HttpServletBean` init方法
```java
  public final void init() throws ServletException {
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Initializing servlet '" + this.getServletName() + "'");
        }

        //获取Servlet init param参数
        PropertyValues pvs = new HttpServletBean.ServletConfigPropertyValues(this.getServletConfig(), this.requiredProperties);
        if (!pvs.isEmpty()) {
            try {
                BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
                ResourceLoader resourceLoader = new ServletContextResourceLoader(this.getServletContext());
                bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, this.getEnvironment()));
                this.initBeanWrapper(bw);
                bw.setPropertyValues(pvs, true);
            } catch (BeansException var4) {
                if (this.logger.isErrorEnabled()) {
                    this.logger.error("Failed to set bean properties on servlet '" + this.getServletName() + "'", var4);
                }

                throw var4;
            }
        }

        // FramworkServlet中执行具体的initServletBean
        this.initServletBean();
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Servlet '" + this.getServletName() + "' configured successfully");
        }

    }
```

```java
protected final void initServletBean() throws ServletException {
        this.getServletContext().log("Initializing Spring FrameworkServlet '" + this.getServletName() + "'");
        if (this.logger.isInfoEnabled()) {
            this.logger.info("FrameworkServlet '" + this.getServletName() + "': initialization started");
        }

        long startTime = System.currentTimeMillis();

        try {
            // 先初始化上下文，在initWebApplicationContext方法中，先拿到ContextLoaderListeners加载过得根上下文，然后通过BeanUtils
            //创建一个子上下文，设置必要的属性，然后再存到ServletContext中去。
            this.webApplicationContext = this.initWebApplicationContext();
            this.initFrameworkServlet();
        } catch (ServletException var5) {
            this.logger.error("Context initialization failed", var5);
            throw var5;
        } catch (RuntimeException var6) {
            this.logger.error("Context initialization failed", var6);
            throw var6;
        }

        if (this.logger.isInfoEnabled()) {
            long elapsedTime = System.currentTimeMillis() - startTime;
            this.logger.info("FrameworkServlet '" + this.getServletName() + "': initialization completed in " + elapsedTime + " ms");
        }

    }

```

DispatcherSerlvet持有一个以自己的Serverlet名称命名的IoC容器。这个IoC容器是一个WebApplicationContext对象，这个IoC容器建立起来后，意味着
DispatchServlet拥有自己的Bean定义空间，这为使用各个独立的XML文件来配置MVC中各个Bean创造了条件，由于初始化结束以后，与Web容器相关的初始化过程实际上已经完成，
Spring MVC 的具体实现，和普通的Spring 应用的程序的实现并没有太大的差别。在Spring MVC DispatchServet的初始化过程中，以对HandlerMapping的初始化调用为触发点
了解，这个调用关系最初是有HttpServletBean的init方法触发的，这个HttpServletBean是HttpServlet的子类。   接着会在HttpServletBean的子类FrameworkServlet中
对IoC容器完成初始化，在这个初始化的方法中，会调用DispatchServlet的initStrategies方法，在这个方法中，启动整个Spring MVC 框架的初始化。

```java
protected void initStrategies(ApplicationContext context) {
        this.initMultipartResolver(context);
        this.initLocaleResolver(context);
        this.initThemeResolver(context);
        this.initHandlerMappings(context);
        this.initHandlerAdapters(context);
        this.initHandlerExceptionResolvers(context);
        this.initRequestToViewNameTranslator(context);
        this.initViewResolvers(context);
        this.initFlashMapManager(context);
    }
```
根据上面的方法名称，很容易理解，以HandlerMappering为例来说明这个initHandlerMappings过程。，这里的Mapping的关系作用是，以Http请求找到相应的Controller控制器，从而
利用这些控制器Controller去完成设计好的数据处理工作，HandlerMappings完成对MVC中Controller的定义和配置。

# HandlerMapping的配置和设计原理
在初始化完成时，在上下文环境中定义的素有HandlerMapping都已经被加载了，这些加载的HandlerMapping 被放在一个List中被排序，存储着Http请求对应的映射数据，这个List中
的每一个元素都对应着一个具体的handlerMapping配置。
以SimpleUrlHandlerMapping这个handlerMapping为例子，分析HandlerMapping的设计与实现，在SimpleHandlerMapping中，定义一个map来持有一系列的映射关系，通过这些映射关系，使Spring
MVC 应用可以根据 Http请求确定一个对应的Controller.