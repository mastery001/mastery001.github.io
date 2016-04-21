---
layout: post
categories: DataStructure
title: 判断两个单向链表是否相交
tags: [Linear table]
---

    1.链表定义
    2.相交的理解
    3.解决方法
        3.1 双重循环判断
        3.2 判断最后一个结点是否相同
        3.3 链表成环

<!--more-->

# 1.链表定义

链式存储结构存储线性表的方法是把存放数据元素的结点用指针域构造成链。指针是指向下一个节点的引用，由数据元素域和一个或若干个指针域组
成的一个类称之为结点。链式存储结构的特点是数据元素间的逻辑关系表现在节点的链接关系上。

# 2.相交的理解

若两个链表`相交`则其相交后的结点必定是最后一个结点或者是第k个结点能使其后面的结点连结成一条直线的结点

则可以得出若两个单向链表相交，则两个链表组成的形状一定为V型或Y型。

# 3.解决方法
    可以考虑用三种方式来实现：
        (1)使用双重循环来遍历结点判断是否相同
        (2)判断两个链表的最后一个结点是否相同
        (3)将两个链表构成环

设定链表结点的数据结构为

    Node{
        DataType data;  //数据元素域
        Node next;      //指针域
    }
第一个结点的引用为root；

## 3.1 双重循环判断
对两个单向链表进行循环，临界值：当第一个链表的结点与第二个链表的结点相同时返回true，反之返回false。

    /**
    *  使用while进行循环迭代，条件为next不为null
    */
    while root1 != null
    begin
        while root2 != null
        begin
            if root1 == root2 then do return true
            end if
        end
    end
    return false

时间复杂度分析： 最好的情况下是root1==root2,此时只循环一次，时间复杂度为O(1),最坏情况下是两个链表的最后
一个结点相等时，循环需要n*n次，即时间复杂度为O(n2).

## 3.2 判断最后一个结点是否相同
得到两个单向链表的最后一个结点，临界值：两个结点相同时返回true，反之返回false

    /**
    *   用root1指代第一个链表的头结点，root2指代第二个链表的头结点
    */
    while root1 != null
    begin
        root1 = root1.next
    end

    while root2 != null
    begin
        root2 = root2.next
    end

    if root1 == root2 then return true
    else 
        return false
    end if

时间复杂度分析： 该算法的时间复杂度在最好和最坏的情况下都是O(n)，都必须循环n次到尾结点。

## 3.3 链表成环
假设两个链表尾部相连能够构成环，则一个链表的首结点遍历到末尾结点必然是另一个链表的首结点。
    
    /**
    *   该程序的先觉条件是必须在两个链表已经构成环的基础上可执行
    */
    while root1 != null
    begin 
        root1 = root1.next
    end
    if root1 == root2 then return true
    return false


时间复杂度分析： 该算法的时间复杂度在最好和最坏的情况下都是O(n)，都必须循环n次到尾结点。
