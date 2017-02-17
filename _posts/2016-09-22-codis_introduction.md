---
layout: post
title: Codis初体验心得
tags: cache
categories: Knowledge
---

Codis是一个分布式的Redis解决方案，对于上层的应用来说，连接Codis Proxy和连接原生的Redis Server没有明显的区别，上层应用可以像使用单机的Redis一样使用，Codis底层会处理请求的转发，不停机的数据迁移等工作，所有后边的一切事情，对于前面客户端来说是透明的，可以简单的认为后边连接是一个内存无限大的Redis服务。
<!--more-->

# 前言

Redis和Memcache是当下最流行的Cache技术，都是可基于内存的Cache，读写效率高。

Redis已经是一个必不可少的部件，丰富的数据结构和超高的性能以及简单的协议，让Redis能够很好的作为数据库的上游缓存层。当然我们的项目中也使用了Redis进行Cache。Redis的优点有许多，这里我就不多做说明，主要列出下列几点Redis不足的地方。

Redis缺点：

1. 耗内存。尽管Redis对一些数据结构采用了压缩算法存储，但占用内存量还是过高。
2. Redis的单点问题。单点Redis容量大小总受限于内存，在业务对性能要求比较高的情况下，单个Redis显然无法满足我们的需求。

对于单点Redis问题我们自然想到进行分布式扩容，目前市面上有类似：

- Redis自带的Cluster（官方不推荐使用）
- Twitter的Twemproxy
- 豌豆荚工程师开发的Codis（和Codis升级后的RebornDB）

至于选择是Twemproxy还是Codis看各个业务自己的需求，目前博主项目使用的就是Codis。

为什么选择Codis：

- 业务需要，数据可能需要迁移，机器横向扩容
- 经过线上测试，Codis的升级版Reborn在pipline操作的性能比Codis慢了几十倍

附上几个链接：

1. [Redis常见集群方案、Codis实践及与Twemproxy比较](http://www.voidcn.com/blog/wuliusir/article/p-5783182.html)
2. [codis,redis,twemproxy三者对比](https://github.com/CodisLabs/codis/issues/309)

# 安装

Codis的安装请参考官方步骤：[Build Codis](https://github.com/CodisLabs/codis/blob/release2.0/doc/tutorial_zh.md#build-codis-proxy--codis-config)

简要步骤：

1. 安装Go
2. 使用Go获取Codis代码
3. Go编译Codis代码

# 部署

Codis的部署可参考官网的步骤：[Codis部署](https://github.com/CodisLabs/codis/blob/release2.0/doc/tutorial_zh.md#部署)

博主这里有一个一键执行的脚本示例大家可以参考：[Codis sample](https://github.com/mastery001/codis-spring-java/tree/master/sample)

**Notes:**脚本中的配置需要改为自己的zookeeper地址和一些Redis服务IP

## config.ini
>作用：基础配置，zookeeper地址以及proxy_id等

1. coordinator：可选择etcd或者zookeeper，示例中的是zookeeper，这里只介绍zookeeper的配置，至于etcd的配置请参考官方文档
2. zk地址：host:ip 示例：zk=192.168.0.123:2181  如果有多个zk，则以逗号`,`分隔
3. product：产品名称, 这个Codis集群的名字, 可以认为是命名空间, 不同命名空间的Codis没有交集 
4. dashboard_addr：dashboard 服务的地址, CLI 的所有命令都依赖于 dashboard 的 RESTful API, 所以必须启动，一般ip配置为当前主机，port默认配置为18087
5. proxy_id：代理的id，不同机器的代理id不能相同

## start_dashboard.sh
>作用：启动dashboard

一般不用修改，但是当config.ini中的dashboard_addr配置的port不是18087时，需要将该shell中的port改为相同的port

## start_redis.sh
>作用：启动Redis Server

示例中启动了四个codis-server(即Redis Server)，开启个数各自行调整，另外Redis的conf使用的是当前目录下redis_conf中的配置，可自行修改

## add_group.sh
>作用：为Redis Server分组

替换配置中的host和port，为codis-server的host和ip。根据业务需要进行合理的分组，可配置多个组

## initslot.sh
>作用：初始化slot

这一步非常重要，为了让缓存均匀的分布到每个codis-server中，在初始化的时候就需要将slot进行均匀分配。

最佳分配公式： 1024 / group个数

如有6个group，则可以按如下分配

```xml
../bin/codis-config -c  config.ini slot range-set 0 170 1 online
../bin/codis-config -c  config.ini slot range-set 171 341 2 online
../bin/codis-config -c  config.ini slot range-set 342 512 3 online
../bin/codis-config -c  config.ini slot range-set 513 693 4 online
../bin/codis-config -c  config.ini slot range-set 694 864 5 online
../bin/codis-config -c  config.ini slot range-set 865 1023 6 online
```

## start_proxy.sh
>作用：启用代理

- --addr：配置访问codis的ip和port
- --http-addr：codis的监控ip和port

完成上面的步骤后即可在浏览器下访问：http://localhost:18087/admin 进行查看codis的状态

Dashboard 
![dashboard](/images/codis/dashboard.png)

Server_group
![server_group](/images/codis/server_group.png)

Slots
![slots](/images/codis/slots.png)

# 常见问题

1. 初始化slot时未全部分配至所有group

这个问题博主在第一次使用的时候遇到过，分配了6个group，但是在初始化slot的时候配置是：

```xml
../bin/codis-config -c  config.ini slot range-set 0 511 1 online
../bin/codis-config -c  config.ini slot range-set 512 1023 2 online
```
只分配了两个group，这样导致key只能存储在前两个group中，另外4个group都不会存储了。

**解决方案**：

- (1)执行Auto Rebalance

在Dashboard界面中找到Migrate Status，点击Auto Rebalance指令，当key的数量特别多且占用内存很大时迁移时间需要很久。
![migrate](/images/codis/migrate.png)

- (2)重新分配slot

具体执行方式请google解决，我尚未尝试过

# For Java Users
>Codis的作者开发了新一代的升级版Codis--Reborn，并为Java开发者提供了相应的[jodis](https://github.com/CodisLabs/jodis)和[Reborn-java](https://github.com/reborndb/reborn-java)

这里博主向大家推荐一个在Reborn-java的基础上对Spring-data-redis的支持的[codis-spring-java](https://github.com/mastery001/codis-spring-java)

- 通过简单的配置即可与Spring-data-redis进行兼容
- 配置zk地址即可自行管理Redis连接池
