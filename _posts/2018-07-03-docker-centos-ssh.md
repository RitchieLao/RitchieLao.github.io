---
layout: post
title: '给Docker里的CentOS加把锁 -- ssh'
subtitle: ''
date: 2018-07-03
categories: Docker
cover: ''
tags: Docker 操作系统
---

> 从 Docker Hub 上拉取下来的 CentOS 没有root密码；这样很不安全！因此，很有必要为其配一把钥匙；确保系统安全。

1. 拉取 CentOS 官方镜像  
	
	```bash 
	# 拉取镜像 
	docker pull centos:7.2.1511 
	
	# 启动并进入 centos
	docker run -it centos:7.2.1511 /bin/bash

	```
	
2. 安装 passwd、openssl、openssh-server 

	```bash  
	 # 使用 yum 安装
	 yum -y install passwd openssl openssh-server

	 
	 # 启动 sshd
	 /usr/sbin/sshd -D

	 # 报错解决
	 ssh-keygen -q -t rsa -b 2048 -f /etc/ssh/ssh_host_rsa_key -N ''  
	 ssh-keygen -q -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N ''
	 ssh-keygen -t dsa -f /etc/ssh/ssh_host_ed25519_key -N ''

	```

3. 修改 sshd_config 配置文件   

	```bash
	 # 编辑 sshd_config
	 vi /etc/ssh/sshd_config
	 # 将以下两行的 yes 改为 no
	 UsePAM no 
	 UsePrivilegeSeparation no
	 
	 # 重启 sshd
	 /usr/sbin/sshd -D

	```

4. 为 root 用户设置密码  

	```bash
	 passwd root
	 root2018 # root密码

	 exit # 退出当前容器

	```  
	
5. 将容器保存为镜像   

	```bash
	# 查看容器的id
	docker ps -a
	CONTAINER ID        IMAGE               COMMAND             CREATED             	STATUS                      PORTS               NAMES
	f3d448d97828        centos:7.2.1511     "/bin/bash"         8 minutes ago       Exited (0) 34 seconds ago                       pedantic_newton

	# 将其保存为docker镜像
	docker commit 3ab474753538 ritchie/centos-ssh
	
	# 删除容器
	docker container rm -f 3ab4


	```

6. 运行测试  

	```bash
	 docker run -d -p 1022:22 ritchie/centos-ssh 

	 # 进入容器
	 ssh root@host-ip -p 1022

	```  


