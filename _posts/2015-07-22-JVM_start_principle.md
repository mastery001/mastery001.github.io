---
layout: post
category: Java
title: JVM启动工作原理
tagline: by Mastery
tags: [Java]
---

    1. JVM体系结构
    2. JVM的生命周期
    3. JVM启动
        3.1 JVM启动代码分析

<!--more-->

# 1. JVM体系结构

    (1)类加载器（`ClassLoader`）（用来装载`.class`文件）

    (2)执行引擎（执行字节码，或者执行本地方法）

    (3)运行时数据区（方法区、堆、java栈、本地方法栈、PC寄存器）


# 2. JVM的生命周期

1.JVM实例对应了一个独立运行的java程序，它是进程级别;
    
    a）启动。启动一个java程序时，一个JVM实例就产生了，任何一个拥有`public static void main(String
    [] args)`方法的class都可以作为JVM实例运行的起点

    b）运行。`main()`作为该程序的初始线程的起点，任何其他线程均由该线程启动。JVM内部有两种线程：守护线
    程和非守护线程，`main()`属于非守护线程守护线程通常由JVM自己使用，java程序也可以标明自己创建的线程
    是守护线程

    c）消亡。当程序中的所有非守护线程都终止时，JVM才退出；若安全管理器允许，程序也可以使用`Runtime`类
    或者`System.exit()`来退出

2.JVM执行引擎实例则对应了属于用户运行程序的线程，它是线程级别的;

# 3. JVM启动

JVM工作原理和特点主要是指操作系统装入JVM，是通过JDK中`java.exe`来完成，通过下面4步来完成JVM环境.

    1.创建`JVM`装载环境和配置

    2.装载`JVM.dll`

    3.初始化`JVM.dll`并挂界到`JNIENV`(`JNI`调用接口)实例

    4.调用`JNIENV`实例装载并处理`class`类

## 3.1 启动代码分析

`main`函数在`openjdk\jdk\src\share\bin\main.c`文件中。简单流程分析如下：

    1.SelectVersion->LocateJRE,定位jre的位置。因为在不同的操作系统上定位jre的方式不同，相关代码就
    和平台相关了，windows相关的一些代码在`openjdk\jdk\src\windows`下，而`linux`和`solaris`相关
    代码在`openjdk\jdk\src\solaris`下。这部分调用到的代码都在`{os}\bin\java_md.c`中。简单的说，
    windows中，LocateJRE要通过查找注册表来定位jre的位置，而Linux下可能需要读取环境变量等。

    2.CreateExecutionEnvironment->GetJVMPath得到jvm的路径，其实就是找到相应的动态链接
    库，具体到linux上，就是`libjvm.so`。

    3.LoadJavaVM，linux上是加载'libjvm.so',windows上是加载`jvm.dll`，导
    出`JNI_CreateJavaVM`、`JNI_GetDefaultJavaVMInitArgs`等函数。

    4.获取classpath。

    5.ContinueInNewThread(&ifn,argc,argv,jarfile,classname,ret);在一个新线程中启动jvm。
    ContinueInNewThread的功能是：调用`GetDefaultJavaJVMInitArgs`(其实就
    调用`JNI_GetDefaultJavaVMInitArgs`)获取虚拟机初始化参数，设定线程栈的大小，
    创建java程序需要的各项参数。然后调用下面函数 `ContinueInNewThread0(JavaMain, 
    threadStackSize, (void*)&args);` ;因为不同操作系统创建线程的方式不同，所以又进
    入os相关的代码，`{os}\bin\java_md.c`中的`int ContinueInNewThread0(JNICALL 
    *continuation)(void *), jlong stack_size,void *args)`函数。
        
        阅读其源码，可知linux上是通过`pthread_create`创建线程，Solaris通过`thr_create`创建线程，
        而`windows`则通过`_beginthreadex`创建线程。
        
        在新线程中调用JavaMain函数，即openjdk\jdk\src\share\bin\main.c中的 `int JNICALL 
        JavaMain(void *_args)`,它负责加载要运行的的java class，并调用class的main方法。
        
        处理步骤是：
        
        1).InitalizeJVM->CreateJavaVM(即通过JNI调用JNI_CreateJavaVM),创建jvm。
        2).LoadMainClass加载main class，并确保main方法的签名是正确的。 
        3).mainID = (*env)->GetStaticMethodID(env, mainClass, "main","([Ljava/lang/String;)V");通过JNI得到main方法的method信息。 
        4).mainArgs = NewPlatformStringArray(env, argv, argc);为main方法准备参数数组。 
        5).(*env)->CallStaticVoidMethod(env, mainClass, mainID, mainArgs);通过JNI调用main方法。 
        6).ret = (*env)->ExceptionOccurred(env) == NULL ? 0 : 1;处理异常 
        7).DetachCurrentThread 
        8).DestroyJavaVM 销毁JVM

>下面以windows的实现进行分析：
>
>首先查找jre路径，Java通过`GetApplicationHome api`来获得当前的`java.exe`绝对路径（例如我电脑上的`e:\program
>\java\jdk\bin\java.exe`），那么它会截取到绝对路径`e:\program\java\jdk`，判断`e:\program\java\jdk\jvm.dll`
>文件是否存在，如果存在就把`e:\program\java\jdk`，作为jre路径；如果不存在则判断`e:\program\java\jdk\jre\jvm.dll`
>是否存在，如果存在则将`e:\program\java\jdk\jre`作为jre路径。如果不存在调用`GetPublicJREHome`查`HKEY_LOCAL
>_MACHINE\Software\JavaSoft\Java Runtime Environment\"当前JRE版本号"\JavaHome`的路径作为jre路径。

