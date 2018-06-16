---
layout: post
title: '[Redis] Mac OS 下安装与配置 Redis'
subtitle: ''
date: 2018-06-16
categories: Mac OS
cover: ''
tags: 操作系统 数据库
---

> Redis 是完全开源免费的，遵守BSD协议，是一个高性能的key-value数据库。

1. 下载及安装  

	```bash
	 # 下载 Redis
	 curl -O http://download.redis.io/releases/redis-4.0.9.tar.gz

	 tar zxf redis-4.0.9.tar.gz # 解压

	 cd redis-4.0.9

	 make test # 编译  

	```

2. 在 /usr/local 目录下，创建 redis 的安装目录  

	```bash
	 sudo mkdir -p /usr/local/redis/bin # redis 命令目录

	 sudo mkdir -p /usr/local/redis/etc # 配置文件目录

	 sudo mkdir -p /usr/local/redis/db # 数据库存放目录

	```

3. 将源码包 src 目录下的 redis-server、redis-cli、redis-benchmark、redis-check-aof、redis-check-rdb 复制到安装目录下的 bin 目录  

	```bash
	 cd src
	 sudo cp redis-server redis-cli redis-benchmark redis-check-aof redis-check-rdb /usr/local/redis/bin/

	```  

4. 将源码包中的 redis.conf 复制到安装目录下的 etc 下，并修改

	```bash
	 # 生成配置文件  
	 cd ..
	 sudo cp redis-4.0.9/redis.conf /usr/local/redis/etc/

	 # 编辑 redis.conf
	 sudo vi /usr/local/redis/etc/redis.conf
	 
	 daemonize yes # 将 'no' 修改为 'yes'

	 dir /usr/local/redis/db # 修改数据库存放目录

	 :x # 保存退出

	 # 启动 redis
	 /usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf
	 
	 # 查看 redis 端口
	 netstat -an | grep :6379 

	```

5. 将 redis 命令添加的系统环境变量

	```bash
	 cd # 切换到当前用户的home目录
	
	 vi .bash_profile  # 编辑 .bash_profile

	 # 在文件末行添加此行
	 export PATH=$PATH:/usr/local/redis/bin

	 :x # 保存退出

	 source .bash_profile # 重载 .bash_profile
	```

6. 设置 redis 开机启动

	```bash
	 # 自己编写一个Redis启动脚本
	 vi io.redis.plist
	 <?xml version="1.0" encoding="UTF-8"?>
	 <!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN"
	        "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
	 <plist version="1.0">
	 <dict>
	        <key>Label</key>
	        <string>io.redis</string>
	        <key>ProgramArguments</key>
	        <array>
	                <string>/usr/local/redis/bin/redis-server</string>
	                <string>/usr/local/redis/etc/redis.conf</string>
	        </array>
	        <key>RunAtLoad</key>
	        <true/>
	        <key>KeepAlive</key>
	        <true/>
	 </dict>
	 </plist>
	 
	 :x # 保存退出
	 
	 # 将该文件拷贝到 /Library/LaunchDaemons 目录下
	 sudo cp io.redis.plist /Library/LaunchDaemons
	 
	 # 载入
	 sudo launchctl load /Library/LaunchDaemons/io.redis.plist
	 
	 reboot # 重启系统
	 
	 netstat -an | grep :6379 # 查看端口

	```
