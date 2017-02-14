---
layout: post
title: Java数据结构（四）：线性表之双向链表
tags:  list
categories: DataStructure
---

java实现简单的双向链表，双向链表也叫双链表，是链表的一种，它的每个数据结点中都有两个指针，分别指向直接后继和直接前驱。 所以，从双向链表中的任意一个结点开始，都可以很方便地访问它的前驱结点和后继结点。 一般我们都构造双向循环链表。

<!--more-->


具体代码可参见：[BidirectionalLinkedList.java](https://github.com/mastery001/study/blob/master/study-datastruct/src/main/java/linear/linked/BidirectionalLinkedList.java)

```
package datastructure.linear.linked;

import datastructure.exception.StructureException;
import datastructure.linear.AbstractList;

/**
 * @Description 双向链表实现
 * @author mastery
 * @Date 2015年6月30日下午9:11:51
 */
public class BidirectionalLinkedList<T> extends AbstractList<T> {

	class Node<T1> {
		/**
		 * 数据域
		 */
		T1 element;

		/**
		 * 指向前驱结点的指针
		 */
		Node<T1> prior;

		/**
		 * 指向后继结点的指针
		 */
		Node<T1> next;

		public Node(Node<T1> next) {
			super();
			this.prior = next;
			this.next = next;
		}

		public Node(T1 element, Node<T1> prior, Node<T1> next) {
			super();
			this.element = element;
			this.prior = prior;
			this.next = next;
		}

	}

	private Node<T> head;

	private Node<T> currentNode;

	public BidirectionalLinkedList() {
		head = currentNode = new Node<T>(null);
		size = 0;
	}

	/**
	 * 得到当前下标对应的结点
	 * 
	 * @param index
	 * @throws StructureException
	 */
	public void indexNodeToCurrent(int index) throws StructureException {
		currentNode = head;
		if (index < -1 || index > size - 1) {
			throw new StructureException("index参数异常！");
		}
		if (index == -1) {
			return;
		}
		currentNode = head.next;
		int j = 0;
		while (currentNode != null && j < index) {
			currentNode = currentNode.next;
			j++;
		}
	}

	@Override
	public void insert(int index, T t) throws StructureException {
		if (index < 0 || index > size) {
			throw new StructureException("index参数异常！");
		}
		// 得到当前下标的上一个结点
		indexNodeToCurrent(index - 1);
		Node<T> insertNode = new Node<T>(t, currentNode, currentNode.next);
		if (currentNode.next != null) {
			// 将新元素生成结点插入到当前结点下
			currentNode.next.prior = insertNode;
		}
		currentNode.next = insertNode;
		size++;
	}

	@Override
	public void delete(int index) throws StructureException {
		if (isEmpty()) {
			throw new StructureException("链表为空");
		}
		if (index < 0 || index > size) {
			throw new StructureException("index参数异常");
		}
		indexNodeToCurrent(index - 1);
		Node<T> twoNextNode = currentNode.next.next;
		if (twoNextNode != null) {
			twoNextNode.prior = currentNode;
		}
		currentNode.next = twoNextNode;
		size--;
	}

	@Override
	public T get(int index) throws StructureException {
		if (isEmpty()) {
			throw new StructureException("链表为空");
		}
		if (index < 0 || index > size) {
			throw new StructureException("index参数异常！");
		}
		indexNodeToCurrent(index);
		return currentNode.element;
	}

}

```