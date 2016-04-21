---
layout: post
title: Java GC机制
tags: [Java,GC]
category: Java
---

<div class="toc"></div>

#存储划分

##垃圾回收算法
-  1、引用计数(Reference Counting)
-  2、标记清除(Mark-Sweep)
-  3、复制(Copying)
-  4、标记-整理(Mark-Compact)
-  5、增量收集(Incremental Collecting)
-  6、分代(Generational Collecting)：基于对对象生命周期分析后得出的垃圾回收算法。把对象分为年轻代、年老代、持久代，对不同生命周期的对象使用不同的算法（上述方式中的一个）进行回收。
jdk从J2SE1.2开始都是使用分代算法。下面我们都具体介绍分代算法。

##分代算法对象存储划分

1.Young(年轻代)
年轻代分为三个区：**一个**Eden区，**两个**Survivor区。

	对象一般是在'Eden'区生成（大对象直接进入老年代）。当`Eden`区满时，还存活的对象将被复制到`Survivor`区（两个中被标记为活Survivor区的一个），当活动Survivor区满时，该区的存活对象将会被复制到非活动Survivor区，此时两个区的标志得到改变（非活动Survivor区变成活动Survivor区，'活动Survivor`区变成`非活动Survivor`区），当新的活动Survivor区也满时，当对象被复制的次数超过`-XX:MaxTenuringThreshold`参数设定的值时还存在则会被复制到老年代。需要注意，Survivor的两个区是对称的，没先后关 系，所以同一个区中可能同时存在从Eden复制过来对象，和从前一个Survivor复制过来的对象，而复制到年老区的只有从第一个Survivor去过来的对象。而且，Survivor区总有一个是空的。 
	
	`-XX:MaxTenuringThreshold`：设置对象在新生代中最大的存活次数，默认为16

	大对象的定义：需要大量连续内存空间的Java对象 例如：byte []bytes = new byte[4 * 1024 * 1024];

- 1.年轻代已经无法存放
- 2. -XX:PretenureSizeThreshold参数：大于这个值的对象直接进入老年代

**意义：避免在Eden区和两个Survivor区之间发生大量的内存拷贝**。

2.Tenured(老年代)

*老年代存放从年轻代存活的对象。一般来说老年代存放的都是生命期较长的对象。*

3.Perm(持久代)

*用于存放静态文件，Java类，方法的字节码。持久代大小通过`-XX:MaxPermSize=`进行设置。 *

#内存申请和对象衰老过程

##内存申请过程

-  1.JVM会试图为相关Java对象在Eden中初始化一块内存区域；
-  2.当Eden空间足够时，内存申请结束。否则到下一步；
-  3.JVM试图释放在Eden中所有不活跃的对象（minor collection），释放后若Eden空间仍然不足以放入新对象，则试图将部分Eden中活跃对象放入Survivor区；
-  4.Survivor区被用来作为Eden及old的中间交换区域，当OLD区空间足够时，Survivor区的对象会被移到Old区，否则会被保留在Survivor区；
-  5.当old区空间不够时，JVM会在old区进行major collection；
-  6.完全垃圾收集后，若Survivor及old区仍然无法存放从Eden复制过来的部分对象，导致JVM无法在Eden区为新对象创建内存区域，则出现"Out of memory错误"；

##对象衰老过程

>对象的衰老的过程就是GC的过程。可通过jstat –gcutil gc log查看GC日志。
>GC有两种类型：Minor GC 和Full GC


| GC类型 | 触发条件 | 产生动作 | 注意 |
| ----- | ----- |  ------ | ------ |
|Minor GC |大多数情况下，对象在新生代Eden区中分配。当Eden区没有足够的空间进行分配时，虚拟机将会发起一次Minor GC。| **(1)**GC后仍然存活的对象复制到活动Survivor区，假设又经过GC则将Eden区和活动Survivor区都移动到非活动Survivor区，并标记为活动Survivor区。**(2)**清空Eden+活动survivor中所有no reference的对象占用的内存将eden+活动survivor中所有存活的对象copy到非活动survivor中一些对象将晋升到old中:非活动survivor放不下的存活次数超过turningthreshold中的重新计算tenuring threshold(serial parallel GC会触发此项)重新调整Eden 和from的大小(parallel GC会触发此项) |全过程暂停应用;是否为多线程处理由具体的GC决定 |
|Full GC | 触发条件请参考以下四点 | 1.清空heap中no reference的对象2.Perm区中已经被卸载的classloader中加载的class信息3.如配置了CollectGenOFirst,则先触发YGC(针对serial GC)4.如配置了ScavengeBeforeFullGC,则先触发YGC(针对serial GC)|全过程暂停应用;是否为多线程处理由具体的GC决定;是否压缩需要看配置的具体GC

Full GC触发条件：

-  1).老年代空间不足：
老年代空间只有在新生代对象复制进来以及直接创建的是大对象才会出现不足的现象，此时执行Full GC，当空间仍不足时，则抛出`java.lang.OutOfMemoryError: Java heap space`

-  2).Perm 空间满：
Perm区存放的为一些class的信息等，当系统中要加载的类、反射的类和调用的方法较多时，Perm区可能被占满，在没有配置为采用CMS GC的情况下将会执行Full GC。当空间仍不足时，则抛出`java.lang.OutOfMemoryError: PermGen space `

-  3).CMS GC时出现`promotion failed`和`concurrent mode failure`:
对于采用CMS进行旧生代GC的程序而言，尤其要注意GC日志中是否有`promotion failed`和`concurrent mode failure`两种状况，当这两种状况出现时可能会触发`Full GC`。
`promotion failed`是在进行Minor GC时，survivor space放不下、对象只能放入旧生代，而此时旧生代也放不下造成的；`concurrent mode failure`是在执行CMS GC的过程中同时有对象要放入旧生代，而此时旧生代空间不足造成的。
应对措施为：增大`survivorspace`、旧生代空间或调低触发并发GC的比率，但在JDK 5.0+、6.0+的版本中有可能会由于JDK的bug29导致CMS在remark完毕后很久才触发sweeping动作。对于这种状况，可通过设置`-XX:CMSMaxAbortablePrecleanTime=5`（单位为ms）来避免。

-  4).统计得到的Minor GC晋升到旧生代的平均大小大于旧生代的剩余空间
这是一个较为复杂的触发情况，Hotspot为了避免由于新生代对象晋升到旧生代导致旧生代空间不足的现象，在进行`Minor GC`时，做了一个判断，如果之前统计所得到的`Minor GC`晋升到旧生代的平均大小大于旧生代的剩余空间，那么就直接触发`Full GC`。
例如程序第一次触发MinorGC后，有6MB的对象晋升到旧生代，那么当下一次Minor GC发生时，首先检查旧生代的剩余空间是否大于`6MB`，如果小于6MB，则执行`Full GC`。
当新生代采用PSGC时，方式稍有不同，PS GC是在Minor GC后也会检查，例如上面的例子中第一次Minor GC后，PS GC会检查此时旧生代的剩余空间是否大于6MB，如小于，则触发对旧生代的回收。

除了以上4种状况外，对于使用RMI来进行RPC或管理的Sun JDK应用而言，默认情况下会一小时执行一次Full GC。可通过在启动时通过- java-Dsun.rmi.dgc.client.gcInterval=3600000来设置Full GC执行的间隔时间或通过-XX:+ DisableExplicitGC来禁止RMI调用System.gc。

对象分配规则

	1.对象优先分配在Eden区，如果Eden区没有足够的空间时，虚拟机执行一次Minor GC。
	2.大对象直接进入老年代（大对象是指需要大量连续内存空间的对象）。这样做的目的是避免在Eden区和两个Survivor区之间发生大量的内存拷贝（新生代采用复制算法收集内存）。
	3.长期存活的对象进入老年代。虚拟机为每个对象定义了一个年龄计数器，如果对象经过了1次Minor GC那么对象会进入Survivor区，之后每经过一次Minor GC那么对象的年龄加1，知道达到阀值对象进入老年区。
	4.动态判断对象的年龄。如果Survivor区中相同年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代。
	5.空间分配担保。每次进行Minor GC时，JVM会计算Survivor区移至老年区的对象的平均大小，如果这个值大于老年区的剩余值大小则进行一次Full GC，如果小于检查HandlePromotionFailure设置，如果true则只进行Monitor GC,如果false则进行Full GC。 

以下附上JVM经典配置：

1. 吞吐量优先的并行收集器 
如上文所述，并行收集器主要以到达一定的吞吐量为目标，适用于科学技术和后台处理等。 
典型配置： 

§ java -Xmx3800m -Xms3800m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:ParallelGCThreads=20 
-XX:+UseParallelGC：选择垃圾收集器为并行收集器。此配置仅对年轻代有效。即上述配置下，年轻代使用并发收集，而年老代仍旧使用串行收集。 
-XX:ParallelGCThreads=20：配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目

相等。 

§ java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:ParallelGCThreads=20 -XX:+UseParallelOldGC 
-XX:+UseParallelOldGC：配置年老代垃圾收集方式为并行收集。JDK6.0支持对年老代并行收集。 

§ java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:MaxGCPauseMillis=100 
-XX:MaxGCPauseMillis=100:设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值。 

§ java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:MaxGCPauseMillis=100 -XX:+UseAdaptiveSizePolicy 
-XX:+UseAdaptiveSizePolicy：设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低相应时间或者收集频率等，此值建议使用并行收集器时，一直打开。 

2. 响应时间优先的并发收集器 
如上文所述，并发收集器主要是保证系统的响应时间，减少垃圾收集时的停顿时间。适用于应用服务器、电信领域等。 
典型配置： 

§ java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:ParallelGCThreads=20 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC 
-XX:+UseConcMarkSweepGC：设置年老代为并发收集。测试中配置这个以后，-XX:NewRatio=4的配置失效了，原因不明。所以，此时年轻代大小最好用-Xmn设置。 
-XX:+UseParNewGC:设置年轻代为并行收集。可与CMS收集同时使用。JDK5.0以上，JVM会根据系统配置自行设置，所以无需再设置此值。 

§ java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseConcMarkSweepGC -XX:CMSFullGCsBeforeCompaction=5 -XX:+UseCMSCompactAtFullCollection 
-XX:CMSFullGCsBeforeCompaction：由于并发收集器不对内存空间进行压缩、整理，所以运行一段时间以后会产生“碎片”，使得运行效率降低。此值设置运行多少次GC以后对内存空间进行压缩、整理。 
-XX:+UseCMSCompactAtFullCollection：打开对年老代的压缩。可能会影响性能，但是可以消除碎片 
