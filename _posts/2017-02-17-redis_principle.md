---
layout: post
title: Redis内存管理和持久化机制
tags:  redis
categories: Knowledge
---

本篇内容参考了多位前辈对Redis的深入分析的Blog和Redis实现原理等书籍，简单描述了Redis的内存管理和持久化机制，列出了其内存结构和编码类型，以及Redis持久化的RDB和AOF两种方式的基本原理
<!--more-->

## 内存管理
Redis是一个基于内存的key-value的数据库，其内存管理是非常重要的；其针对不同操作系统的差异，同时方便自己实现相关的统计函数，封装了不同平台的实现，具体可参阅[深入redis内部--内存管理](http://www.cnblogs.com/davidwang456/p/3504563.html)；

Redis支持多种不同的数据类型，如下：
- String
- List
- Set
- Hash
- Sorted Set

下面就详细来介绍下其数据结构和编码。
### 数据结构
所有的Redis对象都被封装在RedisObject这个结构体当中，如下：

```c
typedef struct redisObject {
    unsigned type,           // 4字节，数据类型(String,List,Set,Hash,Sorted Set)
    unsigned encoding,   // 4字节，编码方式
    unsigned lru,              // 24字节
    int refcount,               // 对象引用计数
    void *ptr                    // 数据具体存储的指向
} robj;
```

数据类型的常用编码方式如下：

| 数据类型 | 常用 | 少量数据 | 特殊情况 |  读 | 写
| ----- | ----- | ----- | ----- |  ----- |  ----- |  
| String | RAW | EMBSTR | INT  | O(1) | O(1)
| List | LinkedList | ZipList |   |  pop:O(1)<br/>lset:O(N) | push:O(1)<br/>lindex:O(N)
| Set | Hash Table | | INTSET(少量整数) | O(1) | O(1)
| Hash | Hash Table | ZipList | | O(1) | O(1)
| Sorted Set | SkipList | ZipList | | zscore:O(1)<br/>zrank:O(logN) | O(logN) 

编码方式介绍：

- RAW：RedisObject的ptr指向名为sds的空间，包含Len和Free头部和buf的实际数据，Free采用了某种预分配（若Len<1M，则Free分配与Len大小一致的空间；若大于Len>=1M，则Free分配1M空间；SDS的长度为Len+Free+buf+1(额外的1字节用于保存空字符)）
- EMBSTR：与RedisObject在连续的一块内存空间，省去了多次内存分配；条件是字符串长度<=39
- INT：字符串的特殊编码方式，若存储的字符串是整数时，则ptr本身会等于该整数，省去了sds的空间开销；实际上Redis在启动时会默认创建10000个RedisObject，代表0-10000的整数
- ZipList(压缩列表)：除了一些标志性字段外用一块类似数组的连续空间来进行存储，缺点是读写时整个压缩列表都需要更改，一般能达到10倍的压缩比。Hash默认值为512，List默认是64
- Hash Table：默认初始大小为4，使用链地址法解决hash冲突；rehash策略：将原来表中的数据rehash并放入新表，之后替换；大量rehash可能会造成服务不可用，因此Redis使用渐进式rehash策略，分批进行

### 过期机制
Redis为了不影响正常的读写操作，一般只会在必要或CPU空闲的时候做过期清理的动作；

1. 必要：一次事件循环结束，进入事件侦听前
2. CPU空闲：系统空闲时做后台定时清理任务（时间限制为25%的CPU时间）；Redis后台清理任务默认100ms执行1次，25%限制是表示25ms用来执行key清理

过期key清理算法：

1. 依次遍历所有db；
2. 从db中随机取得20个key，判断是否过期，若过期，则剔除；
3. 若有5个以上的key的过期，则重复步骤2，否则遍历下一个db
4. 清理过程中若达到了时间限制，则退出清理过程

## 持久化
Redis支持四种持久化方式；如下：

- 定时快照方式(snapshot)[RDB方式]
- 基于语句追加文件的方式(aof)
- 虚拟内存(vm) <font color="red">已被放弃</font>
- Diskstore方式<font color="red">实验阶段</font>

前两种方式为小数据量追加落地方式；后两种为尝试存储数据超过物理内存时，一次性落地方式；

### 定时快照方式(snapshot)
该方式实际是在Redis内部执行一个定时任务，根据redis.conf中配置的save的时间间隔去检查当前数据改变次数和时间是否满足配置，如果满足则从父进程fork(copy-on-write机制)出一个子进程，通过该子进程遍历内存来转换成rdb文件；

```
# Save the DB on disk:
# 设置sedis进行数据库镜像的频率。
# 900秒（15分钟）内至少1个key值改变（则进行数据库保存--持久化）。
# 300秒（5分钟）内至少10个key值改变（则进行数据库保存--持久化）。
# 60秒（1分钟）内至少10000个key值改变（则进行数据库保存--持久化）。
save 900 1
save 300 10
save 60 10000
 
stop-writes-on-bgsave-error yes

# 在进行镜像备份时,是否进行压缩。yes：压缩，但是需要一些cpu的消耗。no：不压缩，需要更多的磁盘空间。
rdbcompression yes
# 一个CRC64的校验就被放在了文件末尾，当存储或者加载rbd文件的时候会有一个10%左右的性能下降，为了达到性能的最大化，你可以关掉这个配置项。
rdbchecksum yes
# 快照的文件名
dbfilename dump.rdb
# 存放快照的目录
dir /var/lib/redis 
```

缺陷：快照只是一段时间的数据的体现，若发生宕机，数据会丢失

### 基于语句追加文件的方式(aof)
类似mysql的binlog方式，每个数据发生改变都会追加至一个log文件中；

```
# 是否开启AOF，默认关闭（no）
appendonly yes
# 指定 AOF 文件名
appendfilename appendonly.aof
# Redis支持三种不同的刷写模式：
# appendfsync always #每次收到写命令就立即强制写入磁盘，是最有保证的完全的持久化，但速度也是最慢的，一般不推荐使用。
appendfsync everysec #每秒钟强制写入磁盘一次，在性能和持久化方面做了很好的折中，是受推荐的方式。
# appendfsync no #完全依赖OS的写入，一般为30秒左右一次，性能最好但是持久化最没有保证，不被推荐。
```

缺陷：追加log文件可能过大，恢复慢；实时写log会影响redis本身性能

## 参考资料
- [Redis内存使用优化与存储](http://www.infoq.com/cn/articles/tq-redis-memory-usage-optimization-storage)
- [Redis持久化 Snapshot和AOF说明](http://www.cnblogs.com/zhoujinyi/archive/2013/05/26/3098508.html)
- [细解Redis内存管理和优化](https://yq.aliyun.com/articles/67122)
- [Redis 设计与实现](http://redisbook.com/)