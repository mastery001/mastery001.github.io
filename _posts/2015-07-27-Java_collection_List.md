---
layout: post
category: Java
title: Java集合类--List篇
tagline: by Mastery
tags: [collections , list]
---

    1. 集合类概述
    2. Collection接口
        2.1 List篇
            2.1.1 List遍历方式
            2.1.2 ArrayList的实现原理
            2.1.3 LinkedList的实现原理
            2.1.4 ArrayList和LinkedList比较
        2.2 List的使用场景

<!--more-->

# 1. 集合类概述

>Java中以一种比数组存储更复杂的方式来存储对象的一组对象---“容器类”，其基本类
型有List、Set和Map，它们被组织在以Collection及Map接口为根的层次结构中，称之为集合框架。


# 2. Collection接口

**Collection的层次结构图**
![](/images/java_collections/collection_interface.jpg)

## 2.1 List篇

    List的特征是其元素以线性的方式的存储，集合中允许放重复元素。
    List接口主要的实现类有：
    - ArrayList() : 代表长度可以动态改变的数组；可以对元素进行随机的访问，向ArrayList
    中插入与删除元素的速度慢。

    - LinkedList() : 采用链表结构实现。其插入和删除的速度快。

### 2.1.1 List遍历方式：

(1)for循环遍历
    
    /**
    *   List中的get(int index)方法支持取指定下标的元素
    */
    1)使用一般for循环
        for(int i = 0 ; i < list.size ; i ++) {
            E obj = list.get(i);
        }
    2)使用增强型for循环
        for(E obj : list) {
        }

(2)使用迭代器（Iterator）

    /**
    *   Collection集合接口继承了Iterator接口，允许集合被迭代
    *   迭代方式如下：
    */
    Iterator<E> it = list.iterator();
    while(it.hasNext()) {
        E obj = it.next();
    }

### 2.1.2 ArrayList实现原理

ArrayList部分源码分析：

    /**
    * Default initial capacity.
    */
    private static final int DEFAULT_CAPACITY = 10;

    // 存放元素的数组
    private transient Object[] elementData;

    /* 构造方法，
    *   @param initialCapacity : 设置数组的初始容量
    */
    public ArrayList(int initialCapacity) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
        this.elementData = new Object[initialCapacity];
    }

从源码可以分析出：ArrayList采用一个Object型对象数组来存放元素，默认初始容量为10。

    public boolean add(E e) {
        // 当添加元素时判断数组容量超出可容纳范围时会自动扩容
        // 具体会调用grow(int minCapacity)方法
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
    
    // 该方法会自动给数组扩大一倍,数组可被扩大的最大容量为Integer.MAX_VALUE 
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        ....省略部分代码 
    }

### 2.1.3 LinkedList实现原理

LinkedList采用双向链表结构来实现的,使用结点来存储数据域和连接前一个和后一个结点.

    static class Node<E> {
        E item;         // 用来保存数据域
        Node<E> next;   // 下一个结点
        Node<E> prev;   // 上一个结点

        ....此处省略
    }

    // 代指第一个结点
    transient Node<E> first;
    
    transient Node<E> last;

    // 这里只做add方法的讲解
    public boolean add(E e) {
        // 链接下一个结点
        linkLast(e);
        return true;
    }

    void linkLast(E e) {
        // 保存当前的最后一个结点
        final Node<E> l = last;
        //创建一个新添加元素的结点，前一个结点为该链表的最后一个结点
        final Node<E> newNode = new Node<E>(l , e , null);
        // 将新结点作为最后一个结点
        last = newNode;
        // 如果未添加前最后一个结点为空，则说明此时链表没有元素，将新结点作为首结点
        // 否则将新结点链接在末尾
        if(l != null) 
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }

LinkedList数据结构是链式结构，具体查阅数据结构相关的书籍。

### 2.1.4 ArrayList和LinkedList比较

ArrayList和LinkedList的区别：

    1.ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。
    2.对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。
    3.对于新增和删除操作add和remove，LinedList比较占优势，因为ArrayList要移动数据。


## 2.2 List使用场景

    如果涉及到“栈”、“队列”、“链表”等操作，应该考虑用List，具体的选择哪个List，根据下面的标准来取舍。
    01) 对于需要快速插入，删除元素，应该使用LinkedList。
    
    02) 对于需要快速随机访问元素，应该使用ArrayList。
    
    03) 对于“单线程环境” 或者 “多线程环境，但List仅仅只会被单个线程操作”，此时应该使用非同步的类(如ArrayList)。
        对于“多线程环境，且List可能同时被多个线程操作”，此时，应该使用同步的类(如Vector)。






