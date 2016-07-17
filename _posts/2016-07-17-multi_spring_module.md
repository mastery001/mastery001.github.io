---
layout: post
title: Maven多module下的Spring启动
tags: Maven Spring
categories: Spring
---

<div class="toc"></div>

<!--more-->

# 项目结构

在实际开发中，往往是用Maven来进行构建项目，一个项目一般都由多个Maven module组成，例如：

* xxx-base(项目启动module，以下为当前module依赖的module)
    - xxx-common
    - xxx-service(spring配置文件，属性properties文件)
    - ...
    
xxx-service的spring配置文件：

```xml
<context:component-scan base-package="com.letv.lepush.interfaces" />

<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations">
        <list>
            <value>classpath*:properties/xxx.properties</value>
        </list>list>
    </property>
</bean>
```

xxx-service的properties文件：

```peoperties
redis.host=${redis.host}
redis.port=${redis.port}
redis.password=${redis.password}

redis.timeout=100000
redis.pool.maxIdle=600
redis.pool.maxTotal=1500

```

# 项目启动

xxx-base的spring配置文件：

1. 报错方式配置

```xml
<context:component-scan base-package="xxx.xxx" />

<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations">
        <list>
            <value>classpath:properties/xxx.properties</value>
        </list>
    </property>
</bean>

<import resource="classpath:xxx-applicationContext.xml"/>
```

异常如下图：

![错误异常](/images/multi_spring_module/exception.png)

2. 解决方案

在存在PropertyPPlaceholderConfigure Bean的每个配置文件中都加上```<property name="ignoreUnresolvablePlaceholders" value="true" />```，如下所示：

```xml
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="ignoreUnresolvablePlaceholders" value="true" />  
    <property name="locations">
        <list>
            <value>classpath:properties/xxx.properties</value>
        </list>
    </property>
</bean>
```

3. **自动扫描spring配置文件**

当一个工程引用的module过多时，那么需要import的spring配置文件就特别多，这时可以将每个module的配置文件都使用相同的命名，例如`applicationContext.xml`，

在项目启动时通过如下代码启动：

```java
ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("classpath*:applicationContext.xml");
```

这样就可以去除掉一大串的import标签了。
