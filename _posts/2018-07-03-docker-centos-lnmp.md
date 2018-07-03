---
layout: post
title: '基于Docker搭建LNMP开发环境'
subtitle: 'LNMP：CentOS 7.2 + NGINX 1.14 + MySQL 5.7 + PHP 7.1'
date: 2018-07-03
categories: Docker
cover: ''
tags: Docker WEB服务器
---

> Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从Apache2.0协议开源。Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上。本次，将带着大家在Docker上，快速部署一套PHP开发环境 -- LNMP。

1. 运行 CentOS  
	
	```bash 
	 # 运行本地 centos 镜像
	 docker run --privileged -d --name=mac-to-centos -p 1022:22 centos-ssh:7.2 /usr/sbin/init /usr/sbin/sshd -D
	
	 ssh root@192.168.1.100 -p 1022 # 登录容器 
	

	```

2. 安装 Nginx 

	```bash  
	 # 使用 yum 安装所需依赖包
	 yum -y install gcc gcc-c++ autoconf pcre pcre-devel make automake
	 yum -y install wget httpd-tools vim initscripts

	 
	 # 配置 Nginx yum 源
	 vi /etc/yum.repos.d/nginx.repo
	 
	[nginx]
	name=nginx repo
	baseurl=http://nginx.org/packages/centos/7/$basearch/
	gpgcheck=0
	enabled=1
	
	:x # 保存退出

	 # 安装 Nginx
	 yum list | grep nginx
	 yum -y install nginx
	 
	 # 确认是否已安装
	 nginx -V
	 nginx -v
	 
	 # 开启 Nginx
	 systemctl start nginx
	 
	 # 查看nginx端口
	 netstat -tunpl|grep 80
	 
	 # 设置 Nginx 开机启动
	 systemctl enable nginx

	```

3. 安装 MySQL   

	```bash
	 # 下载并安装 mysql yum 源
	 wget http://repo.mysql.com/mysql57-community-release-el7-8.noarch.rpm
	 rpm -ivh mysql57-community-release-el7-8.noarch.rpm
	 
	 # 安装
	 yum -y install mysql-server

	```

4. 配置 MySQL  

	```bash
	 # 编辑 mysql 配置文件
	 vi /etc/my.cnf
	
	 # 添加客户端字符集
	 [client]
	 default-character-set=utf8mb4
	
	 [mysqld]
	 character_set_server=utf8mb4 # 设置服务端字符集
	 datadir=/opt/mysql/data # 修改数据库保存路径

	  :x # 保存退出
	 
	  # 启动 mysql 生成临时密码
	  service mysqld start
	  grep "password" /var/log/mysqld.log # 执行后会输出一个临时密码，务必记住
	 
	  # 登录 mysql 修改密码
	  mysql -uroot -p # 登录
	 
	  # 修改密码
	  SET PASSWORD = PASSWORD('new password');
	 
	  ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
	 
	  flush privileges;
	 
	  # 授权 root 用户在所有网段都可以访问mysql服务器，注意生产环境慎用
	  grant all privileges on *.* to root@"%" identified by "new password";
	 
	  grant all privileges on *.* to root@"localhost" identified by "new password";
	 
	  flush privileges;
	 
	  exit;
	 
	  # 设置 MySQL 开机启动
	  systemctl enable mysqld

	```  	

5. 安装 PHP   

	```bash
	 # 下载并安装 PHP yum源
	 rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
	
	 rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
	
	 # 查看可安装扩展
	 yum search php71w
	
	 # 安装 PHP 以及扩展
	 yum --skip-broken -y install php71w-bcmath php71w-cli php71w-common php71w-dba php71w-devel php71w-embedded php71w-enchant php71w-fpm php71w-gd php71w-imap php71w-interbase php71w-intl php71w-ldap php71w-mbstring php71w-mcrypt php71w-mysql php71w-mysqlnd php71w-odbc php71w-opcache php71w-pdo php71w-pdo_dblib php71w-pear php71w-pecl-apcu php71w-pecl-apcu-devel php71w-pecl-geoip php71w-pecl-igbinary php71w-pecl-igbinary-devel php71w-pecl-imagick php71w-pecl-imagick-devel php71w-pecl-libsodium php71w-pecl-memcached php71w-pecl-mongodb php71w-pecl-redis php71w-pecl-xdebug php71w-pgsql php71w-phpdbg php71w-process php71w-pspell php71w-recode php71w-snmp php71w-soap php71w-tidy php71w-xml php71w-xmlrpc

	 php -v # 检查PHP是否安装

	```

6. 配置 PHP、Nginx  

	```bash
	 # 编辑 www.conf 文件
	 vi /etc/php-fpm.d/www.conf
	
	 # 将执行 php-fpm 的用户和组都改为 nginx 
	 user = nginx
	 group = nginx
	
	 :x # 保存退出
	
	 # 编辑 fastcgi_params 文件
	 vi /etc/nginx/fastcgi_params
  	
	 # 在第6行下添加以下这行
	 fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	
	 :x # 保存退出
	
	 # 创建虚拟主机的配置文件存放目录
	 mkdir /etc/nginx/vhost.d
	
	 # 编辑 nginx.conf 文件
	 vi /etc/nginx/nginx.conf

	 # 在 http 模块末尾添加以下这行
	 include /etc/nginx/vhost.d/*.conf;
	 :x # 保存退出

	 # 编辑 default.conf 文件
	 vi /etc/nginx/conf.d/default.conf

	 server {
    	listen       80;
    	server_name  localhost;

        root   /usr/share/nginx/html;
    	index  index.html index.htm index.php; # 添加php文件索引
    	
    	# php文件解析
    	location ~ \.php$ {
       		fastcgi_pass   127.0.0.1:9000;
      		fastcgi_index  index.php;
        	include        fastcgi_params;
    	}
     }
    
     # 重载 nginx 配置文件
     nginx -s reload -c /etc/nginx/nginx.conf
  	
	 # 启动 php-fpm
	 service php-fpm start
	
	 # 设置 php-fpm 开机启动
	 systemctl enable php-fpm
	 
	 # 查看 php-fpm 进程
	 ps -ef | grep php-fpm
	
	```  

7. 退出容器，将容器保存为docker镜像  

	```bash
	 exit # 退出容器
	 
	 # 查看所有容器的信息
	 docker ps -a
	 CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                  NAMES
	 747dc410f927        centos-nginx-mysql   "/usr/sbin/init /usr…"   2 hours ago         Up 2 hours          0.0.0.0:1022->22/tcp   mac-to-centos
	 
	 # 将容器保存为镜像
	 docker commit 747dc410f927 lnmp:v1
	 
	 # 删除容器
	 docker container rm -f 747d
	 

	``` 

8. 运行测试  

	```bash
	 # 启动刚才保存的镜像 lnmp:v1
	 docker run --privileged -d --name=lnmp-dev -p 1022:22 -p 8080:80 -v $PWD/nginx/conf:/etc/nginx/vhost.d -v $PWD/nginx/html:/opt/nginx/html -p 3306:3306 -v $PWD/data/mysql:/opt/mysql/data lnmp:v1 /usr/sbin/init /usr/sbin/sshd -D
	 
	 # 登录容器
	 ssh root@host-ip -p 1022
	 

	``` 

9. php测试

	```
	 # 编写 phpinfo 测试页
	 vi /usr/share/nginx/html/phpinfo.php
	 <?php
	
	 phpinfo();
	
	 :x # 保存退出
	 
	 # curl测试
	 [root@544c33840c4e html]# curl -I http://127.0.0.1/info.php
	 HTTP/1.1 200 OK
	 Server: nginx/1.14.0
	 Date: Tue, 03 Jul 2018 09:24:50 GMT
	 Content-Type: text/html; charset=UTF-8
	 Connection: keep-alive
	 X-Powered-By: PHP/7.1.18

	``` 

10. 总结  
	至此，基于Docker容器的LNMP开发环境已经搭建完毕，我们可以把这个镜像发布到Docker Hub上去。以后，再也不用担心开发环境啦！～！

