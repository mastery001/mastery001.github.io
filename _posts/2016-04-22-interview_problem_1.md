---
layout: post
title: 记面试题目（1）
tags: JAVA Collections DataStructure Thread
categories: Interview
---

一个晚上，一个突然的面试电话呼来，让我措手不及，面试的过程中也发挥的不好，犯了很多不应该犯的错误，一些能答出来的题目却没有答出来。其实真正让我意识到了，工作这段时间自己将很多基础知识都荒废了，而且新知识也并没有接触到多少。

下面就简单将面试的题目总结并解答。

<!--more-->

## Java反射的原理
>Reflection(反射)机制是jdk1.5之后新增的功能，简单来说，反射机制就是在程序运行时能够获取自身的信息；仅知道一个类的全限定类名即可获取该类中的方法，构造器，字段，类型，修饰符等所以信息。

在Java中，创建对象的方式有以下几种

1. 使用`new`关键字
2. 使用`Class.forName(className).newInstance()`
3. 使用cglib动态生成类

无论创建对象的方式是怎样的，其原理都是通过**`类加载器将该类对应的class文件加载到JVM内存区域`**([类加载机制](http://xiaohuishu.net/2015/06/15/%E6%8E%A2%E7%A9%B6JVM%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6/))；并在内存中生成一个代表这个类的java.lang.Class对象，做为这个类的各种数据的访问入口(对于Hotspot虚拟机来讲，Class对象存放在方法区)。

## HashMap原理
>HashMap是Java语言中一种基于散列表(数组+链表)实现的数据结构，其典型的运用了空间换时间的策略，采用hash算法来定位数组下标方式使得查询的时间复杂度最优情况下为O(1).

![hashmap散列结构](/images/interview1/hashmap.png)

HashMap需要注意的几点

- 初始化的容量大小一般为2的n次幂，如果不是，则内部会自动取值为改值为2的n次幂的接近值，例如，当值为15时，自动取值为16.
- HashMap是线程不安全的
- 合理的设置loadFactor参数，能够尽可能的减少[hash冲突](http://www.360doc.com/content/14/0721/09/16319846_395862328.shtml),可参考[hash冲突解决方法](http://blog.csdn.net/lightty/article/details/11191971)

参考资料：

1. [HashMap实现原理分析](http://blog.csdn.net/vking_wang/article/details/14166593)
2. [深入理解HashMap(及hash函数的真正巧妙之处)](http://www.360doc.com/content/10/0505/19/495229_26234886.shtml)
3. [HashMap原理 ](http://blog.chinaunix.net/uid-11775320-id-3143919.html)

### 线程安全的HashMap实现
>HashMap在Java官方文档中是被列为线程不安全的，那么如何自己实现线程安全的HashMap呢？

有以下几种方式：

1. 使用synchronized或java.util.concurrent.locks.Lock对象给get和put操作加锁
2. 使用java.util.concurrent.locks.ReadWriteLock的读写锁，使读写锁分离提高效率

附上方法2的代码：

	import java.util.HashMap;
	import java.util.Map;
	import java.util.concurrent.locks.Lock;
	import java.util.concurrent.locks.ReadWriteLock;
	import java.util.concurrent.locks.ReentrantReadWriteLock;
	
	/**
	 * 线程安全的HashMap
	 * 
	 * @author mastery
	 *
	 *         2016年4月22日 下午3:49:56
	 */
	public class SafeThreadHashMap<K, V> extends HashMap<K, V> implements Map<K, V> {

		/**
		 * 2016年4月22日 下午3:52:04
		 */
		private static final long serialVersionUID = 4530811996637218105L;

		private final ReadWriteLock lock = new ReentrantReadWriteLock();
	
		private final Lock readLock = lock.readLock();
		private final Lock writeLock = lock.writeLock();
	
		public SafeThreadHashMap() {
			super();
		}
	
		public SafeThreadHashMap(int initialCapacity) {
			super(initialCapacity);
		}
	
		@Override
		public V get(Object key) {
			readLock.lock();
			try {
				return super.get(key);
			} finally {
				readLock.unlock();
			}
			
		}
	
		@Override
		public V put(K key, V value) {
			writeLock.lock();
			try {
				return super.put(key, value);
			} finally {
				writeLock.unlock();
			}
			
		}
	}


## 字符逆序问题
>abc,def转换成def,abc，要求空间只有O(1)

这个是一个典型的字符串逆转的题目，面试的时候想到用栈来实现只允许O(1)空间的要求，但是会用到一份临时空间（不确定栈到底需不需要占一份空间）；下面附上解题思路：

-  将被逗号分隔的每个单词先倒序，变成cba,fed
-  再将倒序后的字符串整体倒序，变成def,abc

后来仔细想了下，还是可以不用临时空间的，可以复用字符数组。

代码如下：

	import java.util.Stack;

	public class StringInverse {
	
		private static Stack<Character> stack = new Stack<Character>();
		
		public static String inverse(String str) {
			char[] strArray = str.toCharArray();
			int index =0;
			for(char s : strArray) {
				if(s == ',') {
					while(!stack.empty()) {
						strArray[index++] = stack.pop();
					}
					strArray[index++] = s;
				}else {
					stack.push(s);
				}
			}
			while(!stack.empty()) {
				strArray[index++] = stack.pop();
			}
			for(int i = 0 ; i < index ; i ++) {
				stack.push(strArray[i]);
			}
			index = 0;
			while(!stack.empty()) {
				strArray[index++] = stack.pop();
			}
			return new String(strArray);
		}
		
		public static void main(String[] args) {
			String str = inverse("abc,efg,hij");
			System.out.println(str);
		}
	}

以上的步骤属于个人见解。

## volatile的作用，缺陷

- `可见性`：对一个volatile变量的读，总是能看到(任意线程)对这个volatile变量的写入。意思为工作线程总能第一时间得到主线程的该变量的值。
- `原子性`：对任意单个volatile变量的读/写操作具有原子性，但类似于自加这种复合操作不具有原子性。

有关volatile的解释下面有几篇个人觉得比较好的解释：

-  [深入理解Java内存模型（四）——volatile](http://www.infoq.com/cn/articles/java-memory-model-4/)
-  [java中volatile关键字的含义](http://www.cnblogs.com/aigongsi/archive/2012/04/01/2429166.html)
-  [关于volatile关键字](http://xiaohuishu.net/2015/07/29/%E5%85%B3%E4%BA%8Evolatile%E5%85%B3%E9%94%AE%E5%AD%97/)

***volatile的缺陷：***

volatile的原子性和可见性只是作用于被定义为volatile的变量和简单的对volatile变量的原子操作（例如简单的set和get操作），当对该变量进行运算时则可能会无法保证其特性。


## MySQL索引的实现 ##

附上几篇认为写的蛮好的mysql索引原理的文章：

-  [MySQL索引背后的数据结构及算法原理](http://www.uml.org.cn/sjjm/201107145.asp#nav-2) --详细的介绍了mysql索引的实现，并解释为何使用B树结构
-  [mysql索引结构原理、性能分析与优化](http://www.tuicool.com/articles/ZRN3qu)
-  [Mysql索引的设计、使用和优化](http://c.biancheng.net/cpp/html/1468.html)
