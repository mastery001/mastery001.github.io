---
layout: post
title: 排序算法概括
tags: Sort 
categories: DataStructure
---

排序算法作为数据结构与算法的基础知识，在实际开发中有许多的应用；当然还有更重要的一点，排序算法几乎属于面试必问的知识。博主个人对排序算法做了一些概括，如下文。
<!--more-->

# 引言
 排序总共分为五类：

 1. 插入排序：直接插入排序，希尔排序
 2. 选择排序：直接选择排序，堆排序
 3. 交换排序：冒泡排序，快速排序
 4. 归并排序
 5. 基数排序

 下文将分别用一段话来概述其算法的实现；

# 1. 插入排序
>插入排序是通过构建有序序列的基础上，对于未排序数据从后向前扫描，并找出相应位置插入的过程

## 1.1 直接插入排序
**基本思想**：顺序地将待排序的数据元素按其关键字值的大小插入到已排序数据元素子集合的适当位置。子集合的数据元素个数从只有一个数据元素开始逐次增大，当子集合大小最终与集合大小相同时排序完毕。

通俗地来讲，有一数组a，长度为n,初始第一个元素为集合R{a1}，依次为R{a1,a2}、R{a1,a2,a3}、.....、R{a1,a2,a3,....,an}排序;最终R{a1,a2,a3,....,an}排序的结果即排序完毕；**(Notes:集合顺序不一定是R{a1,a2,a3})**

Java代码详见：[InsertionSort.java](https://github.com/mastery001/study/blob/master/study-datastruct/src/main/java/sort/insert/InsertionSort.java)

```
时间复杂度：
    1. 最佳：O(n)
    2. 平均：O(n^2)
    3. 最坏：O(n^2)
空间复杂度：O(1)
```

## 1.2 希尔插入排序
**基本思想**：把待排序的数据元素分成若干个小组，对同一小组内的数据元素用直接插入法排序；小组的个数逐次缩小，当完成了所有数据元素都在一个组内的排序后排序过程结束。希尔排序又称作为缩小增量排序。

通俗地来讲，有一数组a，长度为n,`并设定一组增量`,并将数组a按增量进行分组，例如有一增量为3，则分组为R{a1,a4,a7.....},R{a2,a5,a8...}等;再将分组好的数组进行插入排序。

Java代码详见：[ShellSort.java](https://github.com/mastery001/study/blob/master/study-datastruct/src/main/java/sort/insert/ShellSort.java)

```
时间复杂度：
    1. 最佳：O(n)
    2. 平均：O((nlog(n))^2)    
    3. 最坏：O((nlog(n))^2)    
空间复杂度：O(1)
```

**增量的选择**：
						
1. 最后一个增量必须为1
2. 应避免增量序列中的取值为倍数（尤其是相邻的值），否则会发生重复比较
						
# 2. 选择排序
>选择排序是每一次从待排序的数据元素中选出最小（或最大）的一个元素，存放在序列的起始位置，直到全部待排序的数据元素排完
						
## 2.1 直接选择排序
**基本思想**：有一数组R{a0,..,an-1}，按顺序从R{a0,..,an-1},R{a1,...,an-1},.......,R{an-2,an-1}中选取出最小值，该值与每次数组中的第一个元素进行交换,总共需要n-1次，得到一个排好序的有序序列。
					
Java代码详见：[SelectSort.java](https://github.com/mastery001/study/blob/master/study-datastruct/src/main/java/sort/select/SelectSort.java)

```
时间复杂度：
    1. 最佳：O(n^2)    
    2. 平均：O(n^2)    
    3. 最坏：O(n^2)    
空间复杂度：O(1)
```

## 2.2 堆排序
**基本思想**：即把`最大堆`堆顶的最大数取出，之后将剩余的堆重新调整为最大堆，并再次将堆顶的最大数取出，直到堆中最后一个数被取出时结束。一般堆中有如下几种操作：

- 最大堆调整（Max-Heapify）：将堆的末端子节点作调整使得子节点永远小于父节点
- 创建最大堆（Build-Max-Heap）：将堆中所有数据重排序，使其成为最大堆
- 堆排序（Heap-Sort）：移除位在最大堆中的堆顶元素，并递归调用最大堆调整运算

堆排序中几个重要的节点计算公式：

1. 父节点：(i-1)/2，i的父节点下标
2. 子节点(左)：2i + 1，i的左子节点下标
3. 子节点(右)：2(i+1)，i的右子节点下标

最大堆与最小堆定义：

- 最大堆(大根堆)：堆顶元素(根节点)的值是堆中所有值中最大值
- 最小堆(小根堆)：堆顶元素(根节点)的值是堆中所有值中最小值


Java代码详见：[HeapSort.java](https://github.com/mastery001/study/blob/master/study-datastruct/src/main/java/sort/select/HeapSort.java)

```
时间复杂度：
    1. 最佳：O(nlog(n))
    2. 平均：O(nlog(n))
    3. 最坏：O(nlog(n))
空间复杂度：O(1)
```
			
# 3. 交换排序
>交换排序是将序列中的值进行比较并按大小进行交换位置；特点为：将值大的数据向序列尾部一定，将值小的数据向序列头部移动
			
## 3.1 冒泡排序
**基本思想**：对每一对相邻元素作比较大小，并按照大小交换位置。

1. 从a0-an数组中，进行相邻元素比较，较大者则与较小者交换位置，这样一次完整比较后则最后一个元素即为最大值
2. 剩余a0-an-1为未排序的序列；重复1）步骤，不过是从a0-an-1数组中进行元素比较
3. 持续每次对越来越少的数组重复上述的步骤，直到没有任何一对元素需要比较
			
Java代码详见：[BubbleSort.java](https://github.com/mastery001/study/blob/master/study-datastruct/src/main/java/sort/convert/BubbleSort.java)
			
```
时间复杂度：
    1. 最佳：O(n)    
    2. 平均：O(n^2)    
    3. 最坏：O(n^2)    
空间复杂度：O(1)
```

## 3.2 快速排序
**基本思想**：快速排序是基于分治法处理的，步骤如下：

1. 分解：将数组a[p,r]划分成两个子数组a[p,q-1]和a[q+1,r],使得A[p,q-1] <= A[q] <= A[q+1,r]
2. 解决：递归调用[分解]步骤，分别对两个子数组进行分解
3. 合并：将最终结果进行合并

简单地说，就是将数组a[p,r]按数组中的某个值(`基准`)划分，小于该值的归为一个数组，大于该值的归为一个数组；之后重复对分解后的数组进行递归操作。
						
Java代码详见：[QuickSort.java](https://github.com/mastery001/study/blob/master/study-datastruct/src/main/java/sort/convert/QuickSort.java)
						
```
时间复杂度：
    1. 最佳：O(nlog(n))
    2. 平均：O(nlog(n))
    3. 最坏：O(n^2)    
空间复杂度：O(n log(n))
```

# 4. 归并排序
**基本思想**：归并排序也是基于分治法处理的；归并排序分为多路归并和两路归并，可用与外排序和内排序；这里主要讲下内排序的两路归并方式。

1. 分解：将数组a[p,r]分为(r-p)/2个长度为2的数组，并使每个数组都有序
2. 合并：将子数组两两合并，最终合成的数据则已经排好序

									
Java代码详见：[MergeSort.java](https://github.com/mastery001/study/blob/master/study-datastruct/src/main/java/sort/other/MergeSort.java)
									
```
时间复杂度：
    1. 最佳：O(nlog(n))
    2. 平均：O(nlog(n))
    3. 最坏：O(n^2)    
空间复杂度：O(n)
```

# 5. 基数排序
**基本思想**：将数组序列中的数(`必须为正整数`)都统一成相同数位，超出位数前面补零；然后从最低位开始，依次按个位、十位、百位....(具体按数的位数决定)的方式进行多次排序，最终则为有序序列；

Java代码详见：[RadixSort.java](https://github.com/mastery001/study/blob/master/study-datastruct/src/main/java/sort/other/RadixSort.java)

```
时间复杂度：
    1. 最佳：O(nk)
    2. 平均：O(nk)
    3. 最坏：O(nk)
空间复杂度：O(n+k)
k为无序序列中最大值的数位10^k，例如98->10^2;故k=2
```

# 总结

本文主要阐述了这些排序算法的基本思想和时间复杂度等情况，并附上了本人写的代码；写此文的初衷亦是为了巩固自己的排序算法基础，如果表述不当的地方请阅读了本文的朋友多指教。谢谢！

附上一些排序算法的不错的blog：

- [http://bubkoo.com/tags/algorithm/](http://bubkoo.com/tags/algorithm/)
- [http://wiki.jikexueyuan.com/project/data-structure-sorting/](http://wiki.jikexueyuan.com/project/data-structure-sorting/)


