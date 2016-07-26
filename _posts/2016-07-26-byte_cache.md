---
layout: post
title: 字节数组的妙用
tags: memory
categories: Java
---

在计算机高级语言中，字节属于最小单位，例如在Java中，int占用4个字节，long占用8个字节等。基本上所有基本类型(包括String)都可以转换成字节，那么这到底有何作用，本篇博客主要是记录了我使用字节数组的经验，希望可以给大家提供一些思路。

<!--more-->

# 缓存对象

## 缓存类型大小分析

在实际开发中，经常会用到本地缓存，或使用`Redis`或者`Memcached`来作分布式缓存,Java一般存入缓存中的对象无非是以下几种:

1. 序列化的Java对象：一个Java对象序列化后所占用的字节是按对象中属性个数，方法个数，以及属性的值决定，最小也需要几百个字节来存储，大的话可能需要几万个字节
2. String(可能是json串)：占用字节由字符串的长度决定
3. 规则的byte[]数组：占用字节由数组长度决定，相比较于String来说，基本类型转换成固定字节的数组，而不是转换成内容长度的String，故字节数组所占用的字节比String更少

 **上述的占用字节数是不考虑Redis或Memcached内部会做出的压缩操作。**

 从上面三种对象分别占用的字节数来分析得出结论：

 1. 在大量的缓存数据(亿级以上)的情况下，为了提高空间利用率，切勿将Java对象当做缓存的内容
 2. 字节数组所需空间最少

## 字节数组的使用实例

>下述场景属于个人虚构的

场景：在电影票系统数据库中，电影院id为int类型，电影id为int类型，上映时间为long类型，电影名称为String类型，现有10亿数据，需要将这些数据存入`Redis`中。

分析：10亿数组采用Java对象作为存储数据显然是不可取，而上映时间为long类型，可能出现数字非常大的id，所以准备采用字节数组形式存储。

实现：

```java
// 缓存结构为：采用hash结构，电影院id为key，电影id为hashKey，value为字节数组，内容为：上映时间+电影名称，存储为大小为8+String.getBytes().length

/**
*	代码只贴出了生成字节数组的部分，至于字节数组转换成值的部分大家可以自行实现
*/
// 以下代码主要获取value，即字节数组
byte[] getBytes(long startTime , String name) {
		byte[] strBytes = name.getBytes();
		int strLength = strBytes.length;
		byte[] cache = new byte[8 + strLength];
		longToByteArray(startTime , cache , 0);
		int index = 8;
		for(int i = 0 ; i < strLength ; i++) {
			cache[index + i] = 	strBytes[i];
		}
}

// 将long类型的数转换成字节数组
void longToByteArray(long value, byte[] byteArray, int start) {
	byteArray[start] = (byte) (value >>> 56);// 取最高8位放到0下标
	byteArray[start + 1] = (byte) (value >>> 48);// 取最高8位放到0下标
	byteArray[start + 2] = (byte) (value >>> 40);// 取最高8位放到0下标
	byteArray[start + 3] = (byte) (value >>> 32);// 取最高8位放到0下标
	byteArray[start + 4] = (byte) (value >>> 24);// 取最高8位放到0下标
	byteArray[start + 5] = (byte) (value >>> 16);// 取次高8为放到1下标
	byteArray[start + 6] = (byte) (value >>> 8); // 取次低8位放到2下标
	byteArray[start + 7] = (byte) (value); // 取最低8位放到3下标
}

// 将字节数组转换成long类型
long byteArrayToLong(byte[] byteArray, int start) {
	byte[] a = new byte[8];
	int i = a.length - 1, j = start + 7;
	for (; i >= 0; i--, j--) {// 从b的尾部(即int值的低位)开始copy数据
		if (j >= start)
			a[i] = byteArray[j];
		else
			a[i] = 0;// 如果b.length不足4,则将高位补0
	}
	// 注意此处和byte数组转换成int的区别在于，下面的转换中要将先将数组中的元素转换成long型再做移位操作，
	// 若直接做位移操作将得不到正确结果，因为Java默认操作数字时，若不加声明会将数字作为int型来对待，此处必须注意。
	long v0 = (long) (a[0] & 0xff) << 56;// &0xff将byte值无差异转成int,避免Java自动类型提升后,会保留高位的符号位
	long v1 = (long) (a[1] & 0xff) << 48;
	long v2 = (long) (a[2] & 0xff) << 40;
	long v3 = (long) (a[3] & 0xff) << 32;
	long v4 = (long) (a[4] & 0xff) << 24;
	long v5 = (long) (a[5] & 0xff) << 16;
	long v6 = (long) (a[6] & 0xff) << 8;
	long v7 = (long) (a[7] & 0xff);
	return v0 + v1 + v2 + v3 + v4 + v5 + v6 + v7;
}

```

***当使用本地缓存需要提高内存的利用率，也是可以使用字节数组的。***

推荐一篇博客：[java Byte和各数据类型(short,int,long,float,double)之间的转换](http://www.2cto.com/kf/201308/235099.html)

## 字节数组的作用

- 提高空间利用率
- 压缩内容，在网络传输时，能有效压缩传输数据的大小，从而提高效率


以上内容纯属个人见解，有不妥的地方请各位大神指正，谢谢！ 
