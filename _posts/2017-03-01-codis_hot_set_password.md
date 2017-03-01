---
layout: post
title: Codis线上加密码
tags: cache
categories: Knowledge
---

<!--more-->

# 线上告警
前面一篇博客已经简单介绍了Codis是什么，具体可跳转：[Codis初体验心得](/2016-09-22/codis_introduction/)

前几日，博主部署的Codis集群的一个group中的master的`Mem Used`显示为`/NaN GB`，
![Mem Used](/images/codis/mem_used.png)
导致Codis集群无法正常提供服务，日志如下：
![error log](/images/codis/error_log.png)

查看该端口的日志发现，

1. 连接zk时出现`request time out`提示错误，
2. 并且在`top`中发现有一个进程占了`778.7%`CPU

![virus](/images/codis/virus.png)

后通过运维了解到，由于redis未设置密码，并且redis存在公网ip，导致黑客通过redis的该漏洞植入了木马程序，导致该端口不能正常提供服务；同样的在网上找到了redis该漏洞的描述：[Redis 未授权访问缺陷可轻易导致系统被黑](http://blog.jobbole.com/94518/)

解决方案：

1. 屏蔽公网ip
2. 给codis集群设置密码

当然这里博主采用的是第二种方案。

# Codis加密
**Notes**：Codis加密的过程中由于要重启代理，所以会导致服务无法正常提供服务；

加密步骤：

1. 在config.ini添加password选项
2. 修改每个codis-server的密码
3. 重启proxy和dashboard
4. 线上服务添加密码并重启

例如本次需要为codis添加密码为`123456`,下面就详细介绍每一步.

## 修改config.ini
在config.ini文件中添加password选项，如下图：
![config](/images/codis/config.png)

## 修改codis-server
每一个codis-server相当于redis服务，所以改其密码可使用与redis方式一致，可参考：[redis配置认证密码](http://blog.csdn.net/zyz511919766/article/details/42268219)

连接每一个codis-server后执行：`config set requirepass 123456`

当然，最好要将对应的redis.conf文件中的requirepass密码也加上哦，这样能保证配置文件与运行的服务一致；

## 重启proxy和dashboard
在描述重启的操作之前先在这引入下`kill`这个指令的作用，这里引用下其描述信息，具体可参考:[kill命令](http://man.linuxde.net/kill)

>kill命令用来删除执行中的程序或工作。kill可将指定的信息送至程序。预设的信息为SIGTERM(15),可将指定程序终止。若仍无法终止该程序，可使用SIGKILL(9)信息尝试强制删除程序。程序或工作的编号可利用ps指令或job指令查看。

通过上面一句话我们可以了解到，kill默认可发送一个信号至程序，预示该程序须终止；那么就意味着当我们调用kill命令关闭codis的程序时，意味着通知zk要移除该codis程序；

所以重启proxy和dashboard如下：

- 获取到proxy和dashboard的进程id
- 调用kill 进程id 通知zk移除proxy和dashboard
- 启动proxy和dashboard；
	1. proxy：`nohup ../bin/codis-proxy --log-level info -c config.ini -L ./log/proxy.log  --cpu=8 --addr=0.0.0.0:19000 --http-addr=0.0.0.0:11000 &`
	2. dashboard：`nohup ../bin/codis-config -c config.ini -L ./log/dashboard.log dashboard --addr=:18087 --http-log=./log/requests.log &>/dev/null &`

## 线上服务添加密码
具体添加密码的方式与redis添加密码方式无异，不过博主这里使用的是经过博主完善的[codis-spring-java](https://github.com/mastery001/codis-spring-java),其基于codis作者提供的[Reborn-java](https://github.com/reborndb/reborn-java)添加了如下功能：

1. 适配Spring-data-redis
2. 当zk连接状态变化时，判断是否CONNECTED或RECONNECTED，若是则重新加载pools
3. 添加password选项

执行完上述步骤后即可上线重启服务.

# 题外话
在本次事故中发现Codis集群在master无法正常提供服务时，slave不会自动切换为master，这是一大缺陷.

后面在网上查阅资料发现有第三方服务[codis-ha](https://github.com/ngaut/codis-ha)，不过貌似其不支持auth，即当codis集群存在密码时会导致主从连接丢失：[dashboard引入codis-ha的功能](https://github.com/CodisLabs/codis/issues/350)


# 参考资料
- [Redis 未授权访问缺陷可轻易导致系统被黑](http://blog.jobbole.com/94518/)
- [redis配置认证密码](http://blog.csdn.net/zyz511919766/article/details/42268219)
