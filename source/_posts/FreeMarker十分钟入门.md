---
title: FreeMarker十分钟入门
date: 2016-01-11
tags: [开源组件]
categories: [十分钟入门系列]
---

在要学习一门新技术直接，一般如何开始入手，达到什么程度叫入门，达到什么程度较学会，达到什么程度叫精通。
学习计算机相关的知识，需要我们时刻保持一颗学习的心态。这里打算写一下我的学习过程，叫做《十分钟入门系列》
当然十分钟并不能保证你完全入门，但是我们更重要的是学习方法。这里第一篇入门系列是以FreeMarker为开篇，因为最近项目中需要用到，而且以前只是听说，所以要用到才会学习，如果你学了，一时半会用不到，那样也很快会忘记。

# 如何学习

直接Google FreeMarker 关键字，为什么不用百度，这个做开发的都知道，百度上的技术文章烂得跟啥一样的。前排一律，文不对题。不适合技术人员寻找资料，如果没办法Google，再怎么着也得用Bing搜索，百度我觉得只是时候查下身边相关的资料，技术文章Google才是归属。

![](http://ww1.sinaimg.cn/large/818b7fe3gy1fii5n5zc3jj218o0wo47k.jpg)

搜索到的结果是前两条是FreeMarker的官网，第三条是国内的CSDN用户写的博客，说明访问量还是挺多的。但是日期是2012年，肯能比较旧了。所以直接略过，学习一手的东西，直接进第一条官网的就OK

![](http://ww1.sinaimg.cn/large/818b7fe3gy1fii5qdkcj1j21yw10sk8u.jpg)
打开FreeMarker官网，直入主题，什么是Apache FreeMarker，看下左侧的目录包括 下载，maven坐标，文档包括手册，java api ，甚至还有中文手册。工具包含编辑器和插件在线模板测试，下面就是组织。
看到这些信息，基本是入门到上手没啥问题了。

# What is Apache FreeMarker ?

读完它的介绍和一副实例图，基本知道FreeMarker是干嘛的了，然后在看完整个介绍后从中获取的信息
    
   1. FreeMarker是一个模板引擎，这里应该想到它和Velocity是同一类东西。
   2. 可以用来生成文本，包括html,邮件，配置文件，源代码等。这里可以知道，一些框架前端现在直接用的FreeMarker来替代jsp，在邮件中实现复杂的html页面将内容动态的替换到指定位置，甚至代码生成器类似的东西也可以用它来做。
   3. 使用FreeMarker模板特有的简单的模板语言FTL,你只需要准备好数据，和模板，然后塞到FreeMarker中，他就可以按照你的模板显示出来。
   4. 遵循MVC设计模式，非常适合动态网页的显示，将页面和java代码分离，设计师可以不用面对嵌套在页面中的逻辑，可以直接修改页面而不用重新编译代码。这里应该想到它和JSP有什么异同，从下面的分析中寻找答案
   5. 不仅可以在web中使用，也可以在非Web环境使用。
   6. 从声明中可以看出FreeMarker项目已经交给Apache开源组织管理，现在还是孵化阶段，后面对孵化阶段做了一个解释。
   7. 特性一 :条件语句，循环，赋值，字符串，和算数运算以及格式化，宏，以及函数，包含其他模板，转义以及其他。
   8. 特性二 :没有任何依赖，各种输出样式，可以从任何地方加载模板。
   9. 特性三 :国际化
   10. 特性四 :XML处理功能
   11. 特性五 :通用数据模型
   
从上面的特性可以知道，FreeMarker支持的功能应该比JSP和Velocity更强大不少。

# Download from Maven

在了解完FreeMarker的基本信息之后，可以从Maven中央仓库中下载它的最新版本来玩一下。最新的是2.3版本
新建一个Web项目测试

# Getting Started 
    
  编写模板文件
```html
<html>
<head>
    <title>Welcome!</title>
</head>
<body>
<h1>Welcome ${user}!</h1>
<p>Our latest product:
    <a href="${latestProduct.url}">${latestProduct.name}</a>!
</body>
</html>
```
如果直接访问该文件，模板中的变量会原样输出到界面上。
   
## 数据模型介绍
数据模型基本结构就是一个对象树，结构和json比较类似。访问方式则和javascript比较类似，子节点的数据类型有字符串，数字，日期/时间 ，布尔值
 
## 模板介绍

 ${...} ftl变量:会被括号中的表达式实际值替换
 
 <#...> ftl标签:也可以称作指令，不会被输出
 
 <@...> ftl自定义标签
 
 <#--...-->  ftl注释，不会被输出
 
 除了以上几个标签其他的任何形式的标签都不会被FreeMarker处理，都会直接输出。
 
 ### 几个基本的指令
   #### if elseif else 指令
    <#if animals.python.price < animals.elephant.price>
      Pythons are cheaper than elephants today.
    <#elseif animals.elephant.price < animals.python.price>
      Elephants are cheaper than pythons today.
    <#else>
      Elephants and pythons cost the same today.
    </#if>
  
    当表达式为true时，会输出包含的内容 当==左右两边的数据类型不匹配时会报错。
    如果表达式是boolean类型，可以使用如下输出
    <#if animals.python.protected>
      Pythons are protected animals!
    </#if>

   #### list 指令
    <table border=1>
      <#list animals as animal>
        <tr><td>${animal.name}<td>${animal.price} Euros
      </#list>
    </table>
    animals是数据模型中的集合 animal是循环出的单个对象，循环体中可以使用ftl变量
    
    如果list是空的，那么上面的会输出一个<table border=1></table>。可以使用将list指令拆成两部分来避免，如下
    
    <#list animals>
        <table border=1>
            <#items as animal>
                <tr><td>${animal.name}<td>${animal.price} Euros
            </#items>
        </table>
    <#list>
    如果animals是空，则里面的所有都不会输出。
    
    
   #### sep指令
     <p>Fruits: <#list misc.fruits as fruit>${fruit}<#sep>, </#list>
     可以按照逗号拼接每个item，最后一个则忽略。
     
   #### include指令
   include指令类似于jsp里面的静态include
   
### 内置插件

    user?upper_case 将user变量的值转为大写
    
    animal.name?cap_first 值转成首字母大写
    
    user?length 获取内容长度
    
    animals?size 获取集合大小
    
    在循环 <#list animals as animal> 标签里面:
    
        animal?index 获取item的索引从0开始
    
        animal?counter 获取item所在位置从1开始
    
        animal?item_parity 返回基数行或者偶数行字符 "odd" or "even"
    
    带有参数的插件
    
    animal.protected?string("Y", "N") 根据表达式的boolean值 返回字符Y或者N
    
    animal?item_cycle('lightRow', 'darkRow') 根据基偶行返回知道的字符
    
    fruits?join(", "): 将集合展开安装指定的字符拼接
    
    user?starts_with("J") 判断表达式的值是否是指定的参数开头，返回true或者false
    
    内置插件可以链式调用
    

### 空值处理
 在Freemarker中一个不存在的变量和该变量值为null是一个意思
 
 为变量指定默认值，当user对象值不存在或为null时，将会用默认值"visitor"代替
 ```html
 <h1>Welcome ${user!"visitor"}!</h1>
 ```
 
 使用表达式??预先判断变量值是否存在然后再渲染
 
```html
<#if user??><h1>Welcome ${user}!</h1></#if>
```

### 特殊字符处理
默认FreeMarker会自动转义所有以$ {...}打印的值，所有的以ftlh和ftlx为扩展名的freemarker模板都会自动关联到HTML和XML输出格式

如果想故意输出特殊字符，则可以使用${value?no_esc}

### 数据类型
   #### 标识符
        字符串，数字，布尔值，日期(日期/时间/日期时间)
   #### 容器
        哈希(对象)，序列(可以使用索引访问)，集合(有限的序列，不能使用索引访问)
   #### 方法和函数
        ${avg(3,5)} 获取平均数
   
        
### 结尾

FreeMarker的文档应该算是非常的详细了，各种错误示例都有。更多的信息可以直接去官网看文档，小白都可以看懂。