---
layout: post
categories: Java
title: JavaClass文件结构
tags: jvm
---
    
    1.Class文件概述
        1.1 无符号数和表
        1.2 魔数和Class文件的版本
        1.3 常量池
            1.3.1 javap的使用
            1.3.2 常量池中的14种常量项的结构总表 
        1.4 访问标志
        1.5 类索引、父类索引和接口索引集合
        1.6 字段表集合
        1.7 方法表集合
        1.8 属性表集合

<!--more-->

# 1.Class文件概述

Class文件是一组以8位字节为基础的二进制流，各个数据项目严格按照顺序紧凑地排列在Class文件之中，中间没有添加任何
分隔符，这使得整个Class文件中存储的内容几乎全部是程序的必要数据，没有空隙存在。当遇到需要占用8位字节的以上的
空间的数据项时，则会按照高位在前的方式分隔成若干个8位字节进行存储。

根据Java虚拟机规范的规定，Class文件格式采用一种类似于C语言结构体的伪结构来存储数据，这种伪结构中只有两种数据类型：
无符号和表，后面的解析都要以这两种数据类型为基础，所以这里要先介绍这两个概念。

## 1.1 无符号数和表

>无符号数属于基本的数据类型，以u1、u2、u4、u8来分别代表1个字节、2个字节、4个字节和8个字节的无符号数，
>无符号数可以用来描述数字、索引引用、数量值或者按照UTF-8编码构成字符串值。
>
>表是由多个无符号数或者其他表作为数据项构成的复合数据类型，所有表都习惯性的以"_info"结尾。表用于描述
>有层次关系的复合结构的数据，整个Class文件本质上就是一张表。它由表6-1所示的数据项构成。

![Class文件格式](/images/java_class/class_file_format.jpg)

## 1.2 魔数与Class文件的版本

>每个Class文件的头4个字节称为魔数（Magic Number），它的唯一作用是确定这个文件是否为一个能被虚拟机接受的Class文件。

Class文件的魔数很有“浪漫气息”，值为`0xCAFEBABY`(咖啡宝贝).

紧接着魔数的4个字节值是Class文件的版本号：第5和第6个字节是次版本号（Minor Version）,第7和第8个字节是主版本号
（Major Version）。Java的版本号是从45开始的。

为了方便讲解，现准备了一段最简单的Java代码（见代码清单1-1）

代码清单1-1

    package org.fenixsoft.clazz;

    public class TestClass {
        
        private int m;
            
        public int inc() {
            return m+1;
        }
    }

**下面是部分class文件中的内容：**

![class file](/images/java_class/clazz_file.jpg)

## 1.3 常量池
>紧接着主次版本号之后的是常量池入口，常量池可以理解为Class文件之中的资源仓库，它是Class文件结构中与其他项目关联
最多的数据类型，也是占用Class文件空间最大的数据项目之一，同时它还是在Class文件中第一个出现的表类型数据项目。

由于常量池常量的数量是不固定的，所以在常量池的入口需要放置一项u2类型的数据，代表常量池容量计数值(constant_pool_count)。

**如下图所示**
![常量池](/images/java_class/class_constant.jpg)

常量池容量（偏移地址：0x00000008）为十六进制数0x0016，即二进制的22.这就代表常量池中有21项常量，索引值范围为1-21.
在Class文件格式规范制定之时，设计者将第0项常量空出来在于满足后面某些指向常量池的索引值的数据在特定情况下需要表达
“不引用任何一个常量池项目”的含义。

    常量池主要存放两大类常量：字面量（Literal）和符号引用（Symbolic References）。字面量比较接近
    Java语言层面的常量概念，如文本字符串,声明为final的常量值等。而符号引用则属于编译原理方面的概念，
    包括了下面三类常量：
     1.类和接口的全限定名（Full Qualified Name）
     2.字段的名称和描述符（Descriptor）
     3.方法的名称和描述符

常量池的每一项常量都是一个表，在JDK1.7之前共有11种结构各不相同的表结构数据，在JDK1.7中为了更好支持动态语言调用，
又额外增加了3种（CONSTANT_MethodHandle_info、CONSTANT_MethodType_info、CONSTANT_InvokeDynamic_info）。

**这14种表都有一个共同特点，就是表开始的第一位是一个u1类型的标志位(tag，取值见下表中标志列)，代表当前常量属于哪种
常量类型.**

![常量池项目类型](/images/java_class/class_constant_type.jpg)

### 1.3.1 javap的使用

>在JDK的bin目录中，有一个专门用于分析Class文件字节码的工具：javap，下图列出了使用javap工具的-verbose参数输出的
TestClass.class文件字节码内容。

![javap输出的字节码内容](/images/java_class/class_javap_info.jpg)

### 1.3.2 常量池中的14种常量项的结构总表

![](/images/java_class/class_constant_struct1.jpg)

![](/images/java_class/class_constant_struct2.jpg)

![](/images/java_class/class_constant_struct3.jpg)

## 1.4 访问标志
>在常量池结束之后，紧接着的两个字节代表访问标志（access_flags），这个标志用于识别一些类或者接口层次的访问信息，
包括：这个Class是类还是接口；是否定义为public类型；是否定义为abstract类型；如果是类的话，是否被声明为final等。

**具体的标志位以及标志的含义见下图**

![access_flags](/images/java_class/class_access_flags.jpg)

access_flags中一共有16个标志位可以使用，上图只定义了其中8个，未使用的标志位要求一律为0.

## 1.5 类索引、父类索引和接口索引集合
>类索引(this_class)和父类索引(super_class)都是一个u2类型的数据，而接口索引集合(interfaces)是一组u2类型的数据的集合
，Class文件中由这三项数据来确定这个类的继承关系。

![](/images/java_class/class_clazz_index.jpg)

## 1.6 字段表集合
>字段表(field_info)用于描述接口或者类中声明的变量。字段(field)包括类级变量和实例级变量，但不包括在方法内部声明的
局部变量。

**字段表结构：**

![field_info](/images/java_class/class_field_info.jpg)

字段修饰符放在access_flags项目中，它与类中的access_flags项目非常类似的，都是一个u2的数据类型，其中可以设置的标志
位的含义见下表：

![class_field_access_flags](/images/java_class/class_field_access_flags.jpg)

跟随access_flags标志的两项索引值：name_index和descriptor_index。它们都是对常量池的引用，分别代表着字段的简单名称
以及字段和方法的描述符。

**名词解释**

    全限定名：将类名中的'.'替换成'/'，例如：java.lang.Object的全限定名为java/lang/Object
   
    简单名称：指没有类型和参数修饰的方法和字段名称，例如Object的equals()方法的简单名称为equals
    
    描述符：用来描述字段的数据类型，方法的参数列表(包括数量、类型以及顺序)和返回值。根据描述符规则
    基本数据类型以及代表无返回值的void类型都用一个大写字符来表示，而对象类型则用字符L加对象的全限
    定名来表示。描述数组类型时，每一维度将使用一个前置的'['字符来描述。

**描述符标识字符含义**

![](/images/java_class/class_field_descriptor.jpg)

    用描述符来描述方法时，按照先参数列表，后返回值的顺序描述，参数列表按照参数的严格顺序放在一组小括号`()`
    之内。如方法void inc()的描述符为`()V`,方法java.lang.String()的描述符为`()Ljava/lang/String;`,
    方法int indexOf(char[] source, int sourceOffset, int sourceCount, char[] target,int targetOffset,int targetCount, int frmoIndex)的描述符为`([CII[CIII)I`

## 1.7 方法表集合
>方法表的结构与字段表的结构相同，依次包括了访问标志(access_flags)、名称索引(name_index)、描述符索引(descriptor_index)
、属性表集合(arrtibutes)几项。

**方法访问标志**

![](/images/java_class/class_method_access_flags.jpg)

**注意事项：** 
    
    如果父类方法在子类中没有被重写(Override)，方法表集合中就不会出现来自父类的方法信息。但同样的
    ，有可能会出现编译器自动添加的方法，最典型的便是类构造器`<clinit>`方法和实例构造器`<init>`方法。

## 1.8 属性表集合
>在Class文件、字段表、方法表都可以携带自己的属性表集合，以用于描述某些场景专有的信息。

**具体属性表在此不作讲解，请参考《[深入理解Java虚拟机：JVM高级特性与最佳实践（最新第二版）](http://yunpan.cn/ccWFtqyhHKAur)》的第六章**

文件提取码是2f30

# 声明

本文摘自《[深入理解Java虚拟机：JVM高级特性与最佳实践（最新第二版）](http://yunpan.cn/ccWFtqyhHKAur)》一书，为了更好的理解java的字节码文件。
