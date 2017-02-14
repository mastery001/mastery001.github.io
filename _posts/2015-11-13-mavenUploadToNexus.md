---
layout: post
category: Experience
title: 使用maven在eclipse中上传code到私服
tagline: by Mastery
tags: [maven , eclipse]
---

最近在公司新开发了一个公用模块，然后需要上传到公司自己的maven私服nexus上，中途遇到蛮多的困难的。所以想在这里总结一下自己的错误，以防以后再犯。

<!--more-->

# 上传至私服的准备工作--配置 #

1.模块pom文件的配置
	
	<build>
		<plugins>
			<!-- 要将源码放上去，需要加入这个插件 -->
			<plugin>
				<artifactId>maven-source-plugin</artifactId>
				<version>2.1.2</version>
				<configuration>
					<attach>true</attach>
				</configuration>
				<executions>
					<execution>
						<phase>compile</phase>
						<goals>
							<goal>jar</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>

	<distributionManagement>
		 <repository>
		    <id>releases</id>
		    <name>releases</name>
		    <url>maven私服releases的url</url>
		 </repository>
		<snapshotRepository>
			<id>snapshots</id>
			<name>snapshots</name>
			<url>maven私服snapshots快照的url</url>
		</snapshotRepository>
	</distributionManagement>

2.settings.xml的配置

需要在<servers></servers>这个区域内加入配置,这个username和password分别是对应releases和snapshots的用户名和密码

	<server>
		<id>releases</id>
		<username>username</username>
		<password>password</password>
	</server>
	<server>
		<id>snapshots</id>
		<username>username</username>
		<password>password</password>
	</server>

3.部署到maven私服

完成上述两步那么准备工作就完成了，之后就可以进行上传部署操作了。这里有两种方式来进行部署。

（1）maven命令行部署

	mvn deploy -X

（2）eclipse下的部署方式

部署如下图：
![eclipse下部署maven代码到nexus私服](/images/maven_to_nexus/maven_to_nexus.jpg)


# 问题总结 #

一般执行deploy -X会打印出出错信息至控制台，报错一般都会返回Return Code,【400、401、402、403、404、405、500、502、503】

## 1. 405错误 ##
问题分析：

405错误的含义是“用来访问本页面的HTTP方法不被允许”，所以这个问题一般是配置上的repository的地址写错了，或者是端口写错了。

解决方案：

检查repository的地址是否写错并改正

## 2. 401或403错误 ##

问题分析：

403错误的含义是“禁止访问”，那么当然是在maven私服上设置了不让该用户访问。

解决方案：

将Releases仓库默认的Deployment Policy修改为“Allow Redeploy”

如果是使用deployment账号登录的朋友请参考
《[maven报错：mvn deploy ）](http://blog.sina.com.cn/s/blog_7833c8450100u2bz.html)》
 
## 3. 400错误 ##

问题分析：

400错误的含义是“错误的请求”，在这里的原因是往往是没有部署到nexus的仓库中。nexus的repository分三种类型：Hosted、 Proxy和Virtual，另外还有一个repository group(仓库组)用于对多个仓库进行组合。部署的时候只能部署到Hosted类型的仓库中，如果是其他类型就会出现这个400错误。

maven的部署是有针对性的，假设只是配置了releases库，但是模块的artifactId中存在SNAPSHOT字样也会报400错误，同理相反也是这种情况。

解决方案：

将releases库和snapshots库两个配置都配置上，或者一一对应的配置。


## 4. 500错误 ##

错误原因：服务器满了

## 5. 402错误 ##

错误原因：你使用的是nexus的Professional版本，但是你的license已经过期了，需要重新注册。


这里只列举出这几种错误，至于其他的错误请参考
《[Maven deploy部署失败原因及解决）](http://blog.csdn.net/kobejayandy/article/details/44282989)》
