---
layout: post
title: Redis scan命令的一次坑
tags: cache
categories: Experience
---

Redis作为当前服务架构不可或缺的Cache，其支持丰富多样的数据结构，Redis在使用中其实也有很多坑，本次博主遇到的坑或许说是Java程序员会遇到的多一点，下面就听博主详细道来。

<!--more-->

# 线上服务堵塞

```
String key = keyOf(appid);
int retryCount = 3;
int socketRetryCount = 3;
Exception ex = null;
while(retryCount > 0 && socketRetryCount > 0) {
	try {
		return redisDao.getMap(key);
	}catch (Exception e) {

	}
}
```

12月2日被告知服务出现异常，查看日志发现其运行到上述代码getMap方法处后日志就没有内容了。

## 问题分析

```
"pool-13-thread-6" prio=10 tid=0x00007f754800e800 nid=0x71b5 waiting on condition [0x00007f758f0ee000]
	java.lang.Thread.State: WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x0000000779b75f40> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
	at org.apache.commons.pool2.impl.LinkedBlockingDeque.takeFirst(LinkedBlockingDeque.java:583)
	at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:442)
	at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:363)
	at redis.clients.util.Pool.getResource(Pool.java:49)
	at redis.clients.jedis.JedisPool.getResource(JedisPool.java:99)
	at org.reborndb.reborn.RoundRobinJedisPool.getResource(RoundRobinJedisPool.java:300)
	at com.le.smartconnect.adapter.spring.RebornConnectionFactory.getConnection(RebornConnectionFactory.java:43)
	at org.springframework.data.redis.core.RedisConnectionUtils.doGetConnection(RedisConnectionUtils.java:128)
	at org.springframework.data.redis.core.RedisConnectionUtils.getConnection(RedisConnectionUtils.java:91)
	at org.springframework.data.redis.core.RedisConnectionUtils.getConnection(RedisConnectionUtils.java:78)
	at xxx.run(xxx.java:80)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:471)
	at java.util.concurrent.FutureTask.run(FutureTask.java:262)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
at java.lang.Thread.run(Thread.java:745)

	Locked ownable synchronizers:
- <0x000000074f529b08> (a java.util.concurrent.ThreadPoolExecutor$Worker)
```

从线程日志可以看出服务堵塞在`获取redis连接`处.

分析：

- 代码配置中redis最大连接为`3000`
- redis配置中`session_max_timeout`为0，即永不断开连接

## 一次修改分析

从以上两点分析得出，redis连接被耗尽，于是查找代码得知由于重写spring-data-redis中的hscan方面导致，代码如下：

```
RedisConnection rc = redisTemplate.getConnectionFactory().getConnection();
if (rc instanceof JedisConnection) {
	JedisConnection JedisConnection = (JedisConnection) rc;
	return new ConvertingCursor<Map.Entry<byte[], byte[]>, Map.Entry<String, String>>(
			JedisConnection.hScan(rawValue(key), cursor, scanOptions),
			new Converter<Map.Entry<byte[], byte[]>, Map.Entry<String, String>>() {

			@Override
			public Entry<String, String> convert(final Entry<byte[], byte[]> source) {

				return new Map.Entry<String, String>() {

				@Override
				public String getKey() {
					return hashKeySerializer.deserialize(source.getKey());
				}

				@Override
				public String getValue() {
					return hashValueSerializer.deserialize(source.getValue());
				}

				@Override
				public String setValue(String value) {
					throw new UnsupportedOperationException(
						"Values cannot be set when scanning through entries.");
				}
			};

		}
	});
} else {
	return hashOps.scan(key, scanOptions);
}
```

上述代码返回ConvertingCursor后未释放连接，导出连接被占满。

## 二次修改分析

于是修改代码为正常释放连接

```
try {
	...
}finally {
	RedisConnectionUtils.releaseConnection(rc, factory);
}
```
代码经过上线，再次跑程序查看线上日志发现报了大量的Connection time out.

于是博主就思考是不是由于重写代码不对，尝试使用spring-data-redis的原生代码，即直接调用`hashOps.scan(key, scanOptions)`方法，再次上线。

上线后观察日志：发现这次不是报`Connection time out`,日志中大量报`Unknown reply: `错误。

分析如下：

由于代码是在多线程环境下运行，有几百个线程去调用hscan操作，spring-data-redis原生的代码执行完一次hscan操作后就会关闭连接并返回一个迭代器Cursor，但是遍历Cursor时在本次count后会再次根据游标重新使用该连接进行查询，可是连接却已经被关闭，这时会使用新的连接是可以正常迭代的，但是一旦复用到其他线程使用的连接则会导致报错`Unknown reply`.

## 三次修改分析

经过思考后得出结论，redis在执行scan操作时一旦连接被释放，那么scan操作将不会进行下去，则报Connection time out. 

查阅官方文档得出结论，redis的scan操作需要full iteration，即最优方式是一个连接将以此scan任务执行完全后释放该连接。

![redis-scan-doc](/images/redis_scan/scan_doc.png)

修改代码如下：

```java
RedisConnectionFactory factory = redisTemplate.getConnectionFactory();
RedisConnection rc = factory.getConnection();
if (rc instanceof JedisConnection) {
	JedisConnection JedisConnection = (JedisConnection) rc;
	Cursor<Map.Entry<String, String>> cursorResult = new ConvertingCursor<Map.Entry<byte[], byte[]>, Map.Entry<String, String>>(
			JedisConnection.hScan(rawValue(key), cursor, scanOptions),
			new Converter<Map.Entry<byte[], byte[]>, Map.Entry<String, String>>() {
			...
			});
return new ScanResult<Map.Entry<String, String>>(cursorResult, factory, rc);}


public void releaseConnection() throws IOException{
	IOException ex = null;
	if(cursor != null) {
		try {
			cursor.close();
		} catch (IOException e) {
			ex = e;
		}
	}
	try {
		RedisConnectionUtils.releaseConnection(rc, factory);
	} catch (Exception e) {

	}
	if(ex != null) {
		throw ex;
	}
}

```
将连接返回给业务代码，并在业务代码执行完毕后将连接释放，问题解决。


# 总结

1. 连接一旦开启就必须释放，否则造成内存泄漏或服务堵塞不可用
2. 重写代码时需要谨记仔细查阅官方文档给出的方案并实施
3. 多线程下使用redis的scan操作需要使用一个连接遍历完Cursor，而不能复用连接，否则导致报错`Unknown reply.`

