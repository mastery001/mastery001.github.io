---
layout: post
title: Java数据结构（三）：线性表之单链表
tags:  list
categories: DataStructure
---

链式存储结构存储线性表的方法是把存放数据元素的结点用指针域构造成链。指针是指向下一个节点的引用，由数据元素域和一个或若干个指针域组成的一个类称之为结点。链式存储结构的特点是数据元素间的逻辑关系表现在节点的链接关系上。

<!--more-->


本例中实现的链表结构都是带头结点的。具体代码如下：【详见：[SingleLinkedList.java](https://github.com/mastery001/study/blob/master/study-datastruct/src/main/java/linear/linked/SingleLinkedList.java)】
```
package datastructure.linear.linked;

import datastructure.exception.StructureException;
import datastructure.linear.AbstractList;

/**
 * @Description 单链表
 * @author mastery
 * @Date 2015年6月30日下午6:11:17
 * @param <T>
 */
public class SingleLinkedList<T> extends AbstractList<T> {

	class Node<T1> {

		T1 element;
		
		Node<T1> next;
		
		public Node(Node<T1> next) {
			super();
			this.next = next;
		}

		public Node(T1 element, Node<T1> next) {
			super();
			this.element = element;
			this.next = next;
		}

		@Override
		public String toString() {
			return element.toString();
		}
	}
	
	
	/**
	 * 头结点,该结点没有参数值，只是指向第一个元素的结点
	 */
	private Node<T> head;
	
	/**
	 * 当前的结点
	 */
	private Node<T> currentNode;
	
	public SingleLinkedList() {
		head = currentNode = new Node<T>(null);
		size = 0;
	}

	/**
	 * 得到当前下标对应的结点
	 * @param index
	 * @throws StructureException
	 */
	public void indexNodeToCurrent(int index) throws StructureException {
		currentNode = head;
		if(index < -1 || index > size -1) {
			throw new StructureException("index参数异常！");
		}
		if(index == -1) {
			return ;
		}
		currentNode = head.next;
		int j = 0 ; 
		while(currentNode != null && j < index) {
			currentNode = currentNode.next;
			j++;
		}
	}
	
	@Override
	public void insert(int index, T t) throws StructureException {
		if(index < 0 || index > size) {
			throw new StructureException("index参数异常！");
		}
		// 得到当前下标的上一个结点
		indexNodeToCurrent(index-1);
		// 将新元素生成结点插入到当前结点下
		currentNode.next = new Node<T>(t , currentNode.next);
		size++;
	}

	@Override
	public void delete(int index) throws StructureException {
		if(isEmpty()) {
			throw new StructureException("链表为空");
		}
		if(index < 0 || index > size) {
			throw new StructureException("index参数异常");
		}
		indexNodeToCurrent(index-1);
		Node<T> twoNextNode = currentNode.next.next;
		currentNode.next = twoNextNode;
		size--;
	}

	@Override
	public T get(int index) throws StructureException {
		if(isEmpty()) {
			throw new StructureException("链表为空");
		}
		if(index < 0 || index > size) {
			throw new StructureException("index参数异常！");
		}
		indexNodeToCurrent(index);
		return currentNode.element;
	}

}

```

*单链表的插入和删除操作的时间效率分析方法和顺序表的插入和删除操作的时间效率分析方法类同，差别是单链表的插入和删除操作不需移动数据元素，只需比较数据元素。*
*要说明的是，虽然单链表插入和删除操作的时间复杂度与顺序表插入和删除操作的时间复杂度相同，但是，顺序表插入和删除操作的时间复杂度指的是移动数据元素的时间复杂度，当数据元素占据的内存空间比较大时，这要比单链表插入和删除操作比较数据元素花费的时间大一个常数倍。另外，单链表求数据元素个数操作的时间复杂度为O(n).*

与顺序表相比，单链表的主要有点是不需要预先确定数据元素的最大个数，插入和删除时不需要移动元素；主要缺点三每个结点中要有一个指针域，因此，空间单元利用效率不高。此外，单链表操作的算法比较复杂。

测试类如下：

```
package datastructure.linear.linked;

import static org.junit.Assert.*;

import org.junit.Test;

import datastructure.exception.StructureException;
import datastructure.linear.List;
import datastructure.linear.linked.SingleLinkedList;

public class SingleLinkedListTest {

	@Test
	public void testSingleLinkedList() throws StructureException {
		List<Integer> list = new SingleLinkedList<Integer>();
		for(int i = 0 ; i < 10 ; i ++) {
			list.insert(i, i+1);
		}
		list.delete(0);
		for(int i = 0 ; i < list.size() ; i++) {
			System.out.print(list.get(i) + "   ");
		}
	}

	@Test
	public void testIndexNodeToCurrent() {
		fail("尚未实现");
	}

	@Test
	public void testInsert() {
		fail("尚未实现");
	}

	@Test
	public void testDelete() {
		fail("尚未实现");
	}

	@Test
	public void testGet() {
		fail("尚未实现");
	}

}

```