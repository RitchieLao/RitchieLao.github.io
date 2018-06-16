---
layout: post
title: '[Redis] Linux下安装与配置 Redis'
subtitle: ''
date: 2018-04-21
categories: linux
cover: ''
tags: Linux 数据库
---

> Redis 是完全开源免费的，遵守BSD协议，是一个高性能的key-value数据库。

1. 下载及安装  

	```bash
	 # 下载 Redis
	 wget http://download.redis.io/releases/redis-4.0.9.tar.gz

	 tar zxf redis-4.0.9.tar.gz # 解压

	 cd redis-4.0.9

	 make # 编译  

	```

2. 在 /usr/local 目录下，创建 redis 的安装目录  

	```bash
	 mkdir -p /usr/local/redis/bin # redis 命令目录

	 mkdir -p /usr/local/redis/etc # 配置文件目录

	 mkdir -p /usr/local/redis/db # 数据库存放目录

	```

3. 将源码包 src 目录下的 redis-server、redis-cli、redis-benchmark、redis-check-aof、redis-check-rdb 复制到安装目录下的 bin 目录  

	```bash
	 cd redis-4.0.9/src
	 cp redis-server redis-cli redis-benchmark redis-check-aof redis-check-rdb /usr/local/redis/bin/

	```  

4. 将源码包中的 redis.conf 复制到安装目录下的 etc 下，并修改

	```bash
	 # 生成配置文件
	 cp redis-4.0.9/redis.conf /usr/local/redis/etc/

	 # 编辑 redis.conf
	 vi /usr/local/redis/etc/redis.conf
	 
	 daemonize yes # 将 'no' 修改为 'yes'

	 dir /usr/local/redis/db # 修改数据库存放目录

	 :x # 保存退出

	 # 启动 redis
	 /usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf

	 netstat -tunpl | grep :6379 # 查看 redis 端口

	 /usr/local/redis/bin/redis-cli

	```

5. 将 redis 命令添加的系统环境变量

	```bash
	 # 编辑 profile
	 vi /etc/profile

	 # 在文件末行添加此行
	 export PATH=$PATH:/usr/local/redis/bin

	 :x # 保存退出

	 source /etc/profile # 重载 profile

	```

6. 设置 redis 开机启动

	```bash
	 # 编辑 rc.local
	 vi /etc/rc.d/rc.local

	 # 在文件末行添加此行
	 /usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf

	 :x # 保存退出

	 reboot # 重启系统

	 netstat -tunpl | grep :6379 # 查看端口

	```
