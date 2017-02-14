---
layout: post
title: Java数据结构（一）：线性表之开篇
tags:  list
categories: DataStructure
---

<!--more-->

**1、线性表**

线性结构的特点是除第一个和最后一个数据元素外的每个数据元素只有一个前驱数据元素和一个后继数据元素。线性表是一个最简单的线性结构。线性表的操作特点主要是可以在任意位置插入和删除一个数据元素。线性表可以用顺序存储结构和链式存储结构存储。用顺序存储结构实现的线性表称作顺序表，用链式存储结构实现的线性表称作链表。链表主要有单链表，循环单链表和双向循环链表三种。顺序表和单链表各有优缺点，并且优缺点刚好相反。

**1.1 线性表的定义**

线性表是一种可以在任意位置进行插入和删除数据元素操作的由n（n>=0）个相同类型数据元素a0,a1,a2,....,an-1组成的线性结构。

**1.2 线性表的接口定义**

该接口名为List<T> 并且使用了泛型来指定对应类型。代码如下：

```
package datastructure.linear;
import datastructure.exception.StructureException;
/**
 * @Description 实现线性表结构的接口
 * @author mastery
 * @Date 2015年6月29日下午8:14:33
 * @param <T>
 */
public interface List<T> {
	/**
	* 在线性表指定位置插入元素t
	* @param index
	* @param t
	*/
	void insert(int index , T t) throws StructureException;
	
	/**
	* 删除该线性表中指定位置的元素
	* @param index
	*/
	void delete(int index) throws StructureException;
	
	/**
	* 获得该线性表中指定位置的元素
	* @param index
	* @return
	*/
	T get(int index) throws StructureException;
	
	/**
	* 得到该线性表当前的存在多少个元素
	* @return
	*/
	int size() throws StructureException;
	
	/**
	* 判断该线性表是否为空
	* @return
	*/
	boolean isEmpty() throws StructureException;
}
```

并且在实现中自定义了一种异常StructureException.代码如下：

```
package datastructure.exception;

public class StructureException extends Exception{

	/**
	 * 
	 */
	private static final long serialVersionUID = 4187679158109073384L;

	public StructureException() {
		super();
	}

	public StructureException(String message, Throwable cause,
			boolean enableSuppression, boolean writableStackTrace) {
		super(message, cause, enableSuppression, writableStackTrace);
	}

	public StructureException(String message, Throwable cause) {
		super(message, cause);
	}

	public StructureException(String message) {
		super(message);
	}

	public StructureException(Throwable cause) {
		super(cause);
	}

}

```

**1.3 线性表的抽象实现**

   该抽象类名为AbstractList<T>：对线性表进行抽象实现，实现了上述接口的两个方法，具体代码如下：
   

```
package datastructure.linear;

import datastructure.exception.StructureException;

public abstract class AbstractList<T> implements List<T>{
	
	/**
	 * 结构的长度
	 */
	protected int size;

	@Override
	public int size() throws StructureException {
		return size;
	}

	@Override
	public boolean isEmpty() throws StructureException {
		return size == 0;
	}
}

```

好了！我们已经定义好接口和抽象类了，下面我们来具体来谈谈实现了吧！