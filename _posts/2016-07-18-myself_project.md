---
layout: post
title: 个人git项目推荐
tags: framework
categories: Project
---

<div class="toc"></div>

学习了Java也有好几年了，在这过程中也开发了几个类似于框架或工具包的项目，在这里就王婆卖瓜一下，觉得还行的话不妨可以Download下来看看。

<!--more-->

# Autumn Framework 

Autumn是基于Servlet的Action Mapping映射、依赖注入和数据库的ORM关系映射实现的JavaWeb框架。

本次设计主要采用了单例模式，工厂模式，适配器模式、模板方法模式、代理模式的组合设计来完成的。

* 采用了Struts的action的基本原理，采用通过配置action（action.xml）的方式来进行前台与控制层的交互。对HttpServletRequest对象进行重写，用于代理request对象的方法进行表单收集操作。并且集成了FileUpload组件的文件上传，Action可通过配置注解自动对表单进行收集成对象！
* 采用了Spring的IOC的思想，通过配置beans.xml文件对dao层和service层对象进行注入，并且内部细节隐藏，可直接通过工厂获取对应的对象！

github地址：[https://github.com/mastery001/Autumn](https://github.com/mastery001/Autumn)

# reactor-http

该项目基于Apache http组件之上进行进一步包装使用，将http请求事件进行垂直切割，以完成http调用监控、统计、降级等作用。

* 0.x版本：适配的是apache http3.1的版本
* 1.x版本：适配的是apache http4.x的版本

github地址：[https://github.com/mastery001/reactor-http](https://github.com/mastery001/reactor-http)

# codis-spring-java

该项目是基于codis作者提供的[Reborn-java](https://github.com/reborndb/reborn-java)添加了如下功能：

* 适配Spring-data-redis
* 当zk连接状态变化时，判断是否CONNECTED或RECONNECTED，若是则重新加载pools
* 添加password选项

github地址：[https://github.com/mastery001/codis-spring-java](https://github.com/mastery001/codis-spring-java)

# Dolphin-Voice-Language

该项目基于openjdk中编译代码(参考了大量的编译代码)逐步实现的解释型语言dv，有助于理解编译原理(词法分析，语法分析)

github地址：[https://github.com/mastery001/Dolphin-Voice-Language](https://github.com/mastery001/Dolphin-Voice-Language)



