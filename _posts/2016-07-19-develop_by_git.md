---
layout: post
title: Git协同开发的那点事
tags: Git
categories: Soft
---

随着分布式和开源这些概念的不断普及，现在有大部分的开发者都在用Git管理开源项目、个人项目和公司的项目开发。本篇博客主要是研究实际在使用Git进行多人协作开发过程中的一些场景和解决方案。

<!--more-->

# 引言

>Git是一个开源的分布式版本控制系统，可以有效、高速的处理从很小到非常大的项目版本管理。

网上有许多介绍Git的博客等，下面就附上链接：

- [Git(百度百科)](http://baike.baidu.com/link?url=Vxrrj2D-qCxXYet_dNwTvM9w7FRyuRWdkQz7zkrzSmGP-C8n5Ql0CwJCtPBWpIjhmAkGVcDficopCEssX2P7zo2eXnbJlrZXH9zRksqTuEe)
- [为什么用Git](http://blog.csdn.net/wengpingbo/article/details/8985132)
- [Git和SVN的区别](http://blog.jobbole.com/31444/)

# 协同开发场景分析

## 场景一：开发分支错误

项目来了个`需求1`，从`develop`分支中拉分支`A`进行开发；开发的中途，又来了一个紧急`需求2`，于是我就直接在`A`分支中开发`需求2`；直到发现不能在`A`分支中开发`需求2`的时候`需求2`的代码已经`commit`了几次了。

如下图提交：

```
m1-m2-m3-m4-m5
```

m1、m2、m5是需求1的代码，m3和m4是需求2的代码

***解决方案：***

**假设m3和m4是commit id为0bda20e和1a04d5f**

利用cherry-pick和revert:

1. 从develop分支创建新分支B：`git checkout B`
2. 将A分支中的需求2的commit复制到分支B：`git cherry-pick 0bda20e 1a04d5f` 
3. 撤销在A分支中开发的需求2：`git checkout A` 和 `git revert 0bda20e 1a04d5f`

附上cherry-pick和revert的一些博客：

- [git cherry-pick](http://yijiebuyi.com/blog/0e65f4a59a1cfa05c5b30ccb6c2f413d.html)
- [git引发的血案（cherry-pick找回丢失的commit)](http://dmouse.iteye.com/blog/1797267)
- [git revert 用法](http://www.cnblogs.com/0616--ataozhijia/p/3709917.html)

## 场景二：分支进度不一致

```xml
m1-m2-m3-m4-m5-m6(master)
       \
       f1-f2-f3-f4-f5(feature)
```

多人协作开发时，你需要开发一个新功能，于是你从`master`上的`commit3`开始拉了一个分支`feature`，可是这个新功能开发周期很长，等到你完成的时候，`master`已经提交到`m6`了。如上图所示。

***解决方案：***

使用rebase：

1. 切换到分支feature：`git checkout feature`
2. rebase到master：`git rebase master`
3. 有冲突解决后执行：`git rebase --continue`

分支树则变成(如下图所示)

```
m1-m2-m3-m4-m5-m6(master)
                \
                f1-f2-f3-f4-f5(feature)
```

附上rebase的一些博客：

- [git rebase简介(基本篇)](http://blog.csdn.net/hudashi/article/details/7664631)
- [git rebase让时光倒流](https://linux.cn/article-4046-1.html)
- [cherry-pick, merge, rebase](http://blog.csdn.net/carolzhang8406/article/details/49761665)
- [git-rebase(认真看，分析很到位)](http://blog.chinaunix.net/uid-27714502-id-3436696.html)

暂时先发这么多，到时再补上。有错误的地方麻烦各位指正。谢谢！
