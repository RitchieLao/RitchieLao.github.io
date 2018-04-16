---
layout: post
title: 'CentOS 6.8 搭建LNMP环境'
subtitle: 'CentOS 6.8 + Nginx 1.10.1 + MYSQL 5.7.11 + PHP 5.6.30'
date: 2018-04-14
categories: Linux
cover: ''
tags: LNMP
---

> LNMP 即：Linux、Nginx、MySQL和PHP的缩写，是当下最流行的WEB服务器架构之一。本文将带领你在CentOS 6.8系统下源码编译一套LNMP环境：    
> 环境参数：  
> 虚拟机：VMware Wokstation 14   
> 系统：CentOS 6.8  
> [ 本教程适用于 CentOS 6.x ]  

### 准备篇
在安装之前，首先要对服务器进行如下设置：  
  
1. 关闭 SELinux、iptables防火墙
	  
	```bash  
	 # 关闭 SELiunx
	 vi /etc/selinux/config
	 SELINUX=disabled
	
	 # iptables 防火墙
	 iptables -F # 清空防火墙
	 service iptables save # 保存防火墙策略
	 
	```   
 
2. 卸载系统自带的 Apache、MySQL、PHP

	```bash
	 # 卸载 Apache
	 rpm -qa | grep httpd* # 查询 httpd
	 httpd-manual-2.2.3-43.e15.centos
	 system-config-httpd-2.2.3-43.e15
	 httpd-2.2.3-43.e15.centos
	
	 #删除 httpd
	 rpm -e httpd-manual-2.2.3-43.e15.centos --nodeps
	 
	```  
	**注意：**重复以上两步，把MySQL和PHP也干掉   

3. 将虚拟机接入真实的网络  
	3.1) 把虚拟机的网卡设置为“桥接模式”  
	3.2) 修改 IP 地址、添加 DNS  
	  
	```bash
	 # 修改 IP 地址  
	 vi /etc/sysconfig/network-scripts/ifcfg-eth0 
	 DEVICE=eth0
	 TYPE=Ethernet
	 ONBOOT=yes
	 BOOTPROTO=static
	 IPADDR=192.168.1.128
	 NETMASK=255.255.0.0
	 GATEWAY=192.168.1.1

	 # 添加 DNS 服务器
	 vi /etc/resolv.conf
	 nameserver 8.8.8.8
	 nameserver 114.114.114.114
	 
	```  
	   
	**注意：**IP地址、DNS，请根据自己的实际环境修改   
	 
4. 用 yum 方式安装编译工具及 LNMMP 环境所需的依赖包  
	4.1) 安装编译工具  
	  
	```bash
	 yum install -y gcc gcc-c++ make unzip cmake prel wget
	 
	```
	  
	4.2) 安装所需的依赖包  
	  
	```bash
	 yum install -y zlib-devel apr* autoconf automake bison bzip2 bzip2* \      
	 cloog-ppl compat* cpp curl curl-devel fontconfig fontconfig-devel \  
	 freetype freetype* freetype-devel gtk+-devel gd gettext \  
	 gettext-devel glibc libcom_err-devel libpng libpng* libpng-devel \  
	 libjpeg* libsepol-devel libselinux-devel libstdc++-devel libtool* \  
	 libgomp libxml2 libxml2-devel libXpm* libX* libtiff libtiff* mpfr \  
	 ntp nasm nasm* openssl-devel patch pcre-devel php-common php-gd \  
	 policycoreutils ppl t1lib t1lib*  libxslt-devel*
	
	```    
	   
5. 搭建 LNMP 环境所需的源码包  
	pcre-8.39.tar.gz　　 ncurses-5.9.tar.gz　　freetype-2.7.tar.gz  
	tiff-4.0.6.tar.gz　　　jpegsrc.v9b.tar.gz　　libevent-1.3.tar.gz    
	yasm-1.3.0.tar.gz　   libgd-2.2.0.tar.gz　　 nginx-1.10.0.tar.gz   
	php-5.6.30.tar.gz　 memcache-3.0.8.tgz　mysql-5.7.11.tar.gz  
	zlib-1.2.8.tar.gz　　boost\_1\_59\_0.tar.gz　　libpng-1.6.25.tar.gz  
	t1lib-5.1.2.tar.gz　 mcrypt-2.6.8.tar.gz　　pecl-memcache-php7.zip  
	mhash-0.9.9.9.tar.gz　libxml2-2.6.30.tar.gz　 libmcrypt-2.5.8.tar.gz  
	openssl-1.0.2j.tar.gz　memcached-1.4.31.tar.gz  
	    
	**下载地址：** [源码包下载 ⬇️](https://pan.baidu.com/s/1lg-j6H2hC1jb-hV61Mk8HQ) 密码：u458 
	  
6. 将源码包用 **WinSPC** 上传至指定目录，并解压  
	6.1) 本次将所有的源码包传到 /lnmp 目录下 (可自定)  
	6.2) 在 /lnmp 目录下编写批量解压文件脚本 "unpack.sh"，提高工作效率  
	  
	```bash
	 vi unpack.sh
	 #!/bin/bash
	 # unpack.sh
	 # 解压各种格式的源码包脚本
	
	 cd /lnmp
	 ls *.tar.gz *.tgz *.tar.bz2 *.zip 2&> /dev/null > list.tmp
	
	 for file in `cat list.tmp`; do
	 	case $file in
	 		*.tar.gz | *.tgz )
	 		 	tar -zxf $file;;
			*.tar.bz2 )
				tar -jxf $file;;
			*.zip )
				unzip -q $file;;
		esac
	 done
	
	 rm -rf *.xml
	
	```  
	6.3) 给 "unpack.sh" 执行权限，运行解压所有源码包  
	  
	```bash
	 chmod a+x ./unpack.sh # 赋予执行权限 x
	 ./unpack.sh # 运行脚本
	
	```

### 安装篇
1. 安装 Nginx  
	1.1) 创建 nginx 用户及用户组   
	 
	```bash
	 groupadd nginx # 创建 nginx 用户组  
	 useradd -r -g nginx nginx # 创建 nginx 用户并加入 nginx 用户组
	 
	```    
	
	1.2) 安装依赖包   
	 
	```bash
	 # 安装 pcre
	 cd /lnmp/pcre-8.39
	 
	 ./configure --prefix=/usr/local/pcre
	 
	 make && make install
	
	 # 安装 openssl
	 cd /lnmp/openssl-1.0.2j
	 
	 ./config --prefix=/usr/local/openssl
	 
	 make && make install
	 
	 # 编辑 profile 
	 vi /etc/profile
	 
	 # 在文件末行添加一下这行，保存退出
	 export PATH=$PATH:/usr/local/openssl/bin
	 
	 source /etc/profile # 重新初始化，使之生效
	 
	```   
	 
	1.3) 安装 Nginx  
	  
	```bash
	 cd /lnmp/nginx-1.10.1
	 
	 ./configure \
	 --prefix=/usr/local/nginx \
	 --without-http_memcached_module \
	 --user=nginx --group=nginx \
	 --with-http_stub_status_module \
	 --with-http_ssl_module \
	 --with-http_gzip_static_module \
	 --with-openssl=/lnmp/openssl-1.0.2j \
	 --with-zlib=/lnmp/zlib-1.2.8 \
	 --with-pcre=/lnmp/pcre-8.39
	
	 make && make install
	
	```   
	 
	 **注意：**openssl 、 zlib 、 pcre 扩展模块指向源码包的路径  
	   
	1.4) 编辑 nginx 配置文件 nginx.conf  
	  
	```bash  
	vi /usr/local/nginx/conf/nginx.conf  
	
    # 修改nginx的执行用户与组为 'nginx'
	user nginx nginx;
	
	# 指定错误日志的保存路径
	error_log logs/nginx_error.log;
	
	# 指定 nginx.pid 的路径
	pid /usr/local/nginx/nginx.pid;
	
	# 修改打开文件最大数为 65535
	worker_rlimit_nofile 65535;
	
	events {
		use epoll;
		
		# 修改最大连接数为 65535
		worker_connections 65535;
	}
	
	server {
		listen 80;
		server_name localhost;
	
		# 指定日志保存路径及日志类型
		access_log logs/localhost.access.log main;
		location / {
			root html;
	
			# 添加 index.php
			index index.html index.htm index.php;
		}
	
		location ~ \.php$ {
			root html;
			fastcgi_pass 127.0.0.1:9000;
			fastcgi_index index.php;
			include fastcgi_params;
		}
	}
	
	```   
	1.5) 启动&测试   
	  
	```bash   
	　/usr/local/nginx/sbin/nginx # 启动 Nginx  
	　  
	　netstat -tunpl | grep :80 # 查看 Nginx 端口 
	　 
	　pstree | grep nginx # 查看 Nginx 服务进程 
	　  
	```  
	
	在浏览器中输入 http://192.168.1.128 成功如下图：  
	![Nginx success](http://p704q21lb.bkt.clouddn.com/2018-04-15-nginx-success.png)   
	 
	1.6) 设置 Nginx 开机启动  
	  
	```bash  
	 # 编辑 /etc/rc.local 文件
	 vi /etc/rc.local
	 
	 # 在文件末行添加以下这行
	 /usr/local/nginx/sbin/nginx &> /dev/null
	 
	```
	
2. 安装 MySQL   
	2.1) 创建用户、安装目录  
	  
	```bash 
	 groupadd mysql # 创建 mysql 组
	 useradd -r -g mysql mysql # 创建 mysql 用户
	 
	 # 创建 mysql 安装目录及数据库目录
	 mkdir -p /usr/local/mysql/data 
	 
	```  
	2.2) 安装依赖包  
	  
	```bash 
	 # 安装 ncurses
	 cd /lnmp/ncurses-5.9
	 
	 ./configure --with-shared \
	 --without-debug \
	 --without-ada \
	 --enable-overwrite
	
	 make && make install
	 
	```  
	 
	2.3) 安装 MySQL   
	 
	```bash 
	 # 安装 mysql
	 cd /lnmp/mysql-5.7.11
	 
	 cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
	 -DMYSQL_DATADIR=/usr/local/mysql/data/\
	 -DDOWNLOAD_BOOST=1 \
	 -DWITH_BOOST=../boost_1_59_0 \
	 -DSYSCONFDIR=/etc \
	 -DWITH_PARTITION_STORAGE_ENGINE=1 \
	 -DWITH_FEDERATED_STORAGE_ENGINE=1 \
	 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
	 -DWITH_INNOBASE_STORAGE_ENGINE=1 \
	 -DWITH_MYISAM_STORAGE_ENGINE=1 \
	 -DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
	 -DMYSQL_TCP_PORT=3306 \
	 -DENABLED_LOCAL_INFILE=1 \
	 -DENABLE_DTRACE=0 \
	 -DEXTRA_CHARSETS=all \
	 -DDEFAULT_CHARSET=utf8 \
	 -DDEFAULT_COLLATION=utf8_general_ci \
	 -DMYSQL_USER=mysql \
	 -DWITH_EMBEDDED_SERVER=1
	
	  make && make install
	  
	```
	
	**注意：**MySQL 5.7以上的版本，安装编译时间有点长，大约需要：40～60分钟左右  

	2.4) ACL 目录授权，使 mysql 用户对/usr/local/mysql 目录拥有所有权限  
	  
	```bash  
	 # 设置mysql用户对/usr/local/mysql目录有可读、可写、可执行权限  
	 setfacl -m u:mysql:rwx -R /usr/local/mysql
	 
	 # 设置默认权限
	 setfacl -m d:u:mysql:rwx -R /usr/local/mysql
	 
	```  
	  
	2.5) 生成 MySQL 配置文件 my.cnf  
	  
	```bash  
	 cp /usr/local/mysql/support-files/my-default.cnf /etc/my.cnf
	 
	 # 修改 /etc/my.cnf 配置文件
	 vi /ect/my.cnf
	 
	 # 在 [mysqld] 配置项下，添加以下两行，指定mysql的安装目录和数据库目录
	 [mysqld]
	 basedir = /usr/local/mysql/
	 datadir = /usr/local/mysql/data
	 
	```  
	2.6) 初始化 MySQL 数据库（不生成随机密码） 
	  
	```bash
	 /usr/local/mysql/bin/mysqld --initialize-insecure --user=mysql --
basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data/ 

	```  
	    
	2.7) 设置 MySQL 开机启动  
	  
	```bash
	 # 在mysql的安装目录中找到mysql的启动脚本mysql.server，将其复制到
	 # /etc/init.d 目录下，并改名为 mysqld
	 cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
	
	 # 给 mysqld 执行权限
	 chmod a+x /etc/init.d/mysqld
	
	 # 将 mysqld 纳入 chkconfig 管理体系
	 chkconfig --add mysqld
	 chkconfig mysqld on
	
	 service mysqld start # 启动 MySQL 
	
	 netstat -tunpl | grep :3306 # 查看 MySQL 端口 
	 
	```  
	  
	2.8) 为 MySQL 管理员账户 root 密码   
	  
	```bash
	 # 初次登录为空密码，回车可直接进入
	 /usr/local/mysql/bin/mysql -uroot -p
	
	 # 设置密码
	 SET PASSWORD FOR 'root'@localhost = PASSWORD('root123');
	
	 # 更新数据库，使密码生效
	 flush privileges;
	 
	``` 
 
3. 安装 PHP  
	3.1) 安装依赖包  
	  
	```bash  
	 # 安装 libxml2
	 cd /lnmp/libxml2-2.6.30
	 
	 ./configure --prefix=/usr/local/libxml2/

	 make && make install

	 # 安装 libmcrypt
	 cd /lnmp/libmcrypt-2.5.8
	 
	 ./configure --prefix=/usr/local/libmcrypt/

	 make && make install

	 # 安装 libltdl
	 cd /lnmp/libmcrypt-2.5.8/libltdl
	 
	 ./configure --enable-ltdl-install

	 make && make install
	
	 # 设置环境变量
	 LD_LIBRARY_PATH=/usr/local/libmcrypt/lib:/usr/local/lib64 

	 # 安装 libpng
	 cd /lnmp/libpng-1.6.25  
	 
	 ./configure --prefix=/usr/local/libpng/

	 make && make install
	 
	 # 安装 freetype
	 cd /lnmp/freetype-2.7
	 
	 ./configure --prefix=/usr/local/freetype/
	
	 make && make install
	
	 # 安装 yasm
	 cd /lnmp/yasm-1.3.0
	 
	 ./configure 
	
	 make && make install
	
	 # 安装 mhash
	 cd /lnmp/mhash-0.9.9.9
	 
	 ./configure 
	
	 make && make install
	 
	 # 生成软链接
	 ln -s /usr/local/lib/libmcrypt.la /usr/lib64/libmcrypt.la
	 ln -s /usr/local/lib/libmcrypt.so /usr/lib64/libmcrypt.so
	 ln -s /usr/local/lib/libmcrypt.so.4 /usr/lib64/libmcrypt.so.4
	 ln -s /usr/local/lib/libmcrypt.so.4.4.8 /usr/lib64/libmcrypt.so.4.4.8
	 ln -s /usr/local/lib/libmhash.a /usr/lib64/libmhash.a
	 ln -s /usr/local/lib/libmhash.la /usr/lib64/libmhash.la
	 ln -s /usr/local/lib/libmhash.so /usr/lib64/libmhash.so
	 ln -s /usr/local/lib/libmhash.so.2 /usr/lib64/libmhash.so.2
	 ln -s /usr/local/lib/libmhash.so.2.0.1 /usr/lib64/libmhash.so.2.0.1
	 ln -s /usr/local/bin/libmcrypt-config /usr/bin/libmcrypt-config 
	 
	 # 安装 mcrypt
	 cd /lnmp/mcrypt-2.6.8
	
	 export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
	
	 ./configure --with-libmcrypt-prefix=/usr/local/libmcrypt
	
	 make && make install
	
	 # 安装 jpegsrc.v9b
	 cd /lnmp/jpeg-9b
	
	 ./configure --prefix=/usr/local/jpeg/ \
	 --enable-shared \
	 --enable-static
	
	 make && make install
	
	 # 安装 zlib
	 cd /lnmp/zlib-1.2.8
	
	 ./configure 
	
	 make && make install
	
	 # 安装 tiff
	 cd /lnmp/tiff-4.0.6
	
	 ./configure --prefix=/usr/local/tiff --enable-shared
	
	 make && make install
	 
	 # 安装 libgd
	 cd /lnmp/libgd-2.2.0
	
	 ./configure --prefix=/usr/local/gd2/ \
	 --with-jpeg=/usr/local/jpeg/ \
	 --with-freetype=/usr/local/freetype/ \
	 --with-fontconfig=/usr/local/freetype/ \
	 --with-xpm=/usr/lib64/ \
	 --with-png=/usr/local/libpng/ \
	 --with-tiff=/usr/local/tiff/ \
	 --enable-shared
	
	 make && make install
	
	 # 安装 t1lib
	 cd /lnmp/t1lib-5.1.2
	
	 ./configure --prefix=/usr/local/t1lib --enable-shared
	
	 make without_doc && make install
	 
	```   
	 
	3.2) 安装 PHP  
	  
	```bash  
	 cd /lnmp/php-5.6.30	

	 yum -y install curl-devel libxslt-devel*
	
	 ./configure --prefix=/usr/local/php \
	 --with-config-file-path=/usr/local/php/etc \
	 --with-curl --with-freetype-dir=/usr/local/freetype \
	 --with-gd=/usr/local/gd2 --with-gettext \
	 --with-iconv-dir --with-mysqli --with-pcre-regex \
	 --with-pdo-mysql --with-pdo-sqlite --with-pear \
	 --with-png-dir=/usr/local/libpng \
	 --with-jpeg-dir=/usr/local/jpeg \
	 --with-openssl=/usr/local/openssl \
	 --with-xpm-dir=/usr/lib64/ \
	 --with-fpm-user=nginx --with-fpm-group=nginx --enable-fpm \
	 --with-xsl --with-zlib --enable-bcmath --enable-libxml \
	 --enable-inline-optimization --enable-gd-native-ttf \
	 --enable-mbregex --enable-mbstring \
	 --enable-opcache --enable-pcntl \
	 --enable-shmop --enable-soap \
	 --enable-sockets --enable-sysvsem \
	 --enable-xml --enable-zip 
	
	 make && make install

	```  
	  
	3.3) 生成配置文件 
	 
	```bash
	 # 生成php配置文件 php.ini
	 cp /lnmp/php-5.6.30/php.ini-production /usr/local/php/etc/php.ini
	
	 # 生成 php-fpm.conf 配置文件
	 cp /usr/local/php/etc/php-fpm.conf.default  /usr/local/php/etc/php-fpm.conf
	
	 # 生成www.conf配置文件
	 cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf
	 
	 # 编辑 php-fpm.con
	 vi /usr/local/php/etc/php-fpm.conf
	 pid = run/php-fpm.pid #去掉行首的';'
	
	 # 修改 /usr/local/nginx/conf/ 目录中的 fastcgi_params 文件 
	 vi /usr/local/nginx/conf/fastcgi_params
	
	 # 在第6行下添加这行
	 fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	 
	```   
	  
	3.4) 设置 PHP 开机启动  
	  
	```bash
	 # 在php源码目录中找到php的启动脚本init.d.php-fpm，将其复制到
	 # /etc/init.d 目录下，并改名为 php-fpm
	 cp /lnmp/php-5.6.30/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
	
	 # 给 php-fpm 执行权限
	 chmod a+x /etc/init.d/php-fpm
	
	 # 将 php-fpm 纳入 chkconfig 管理体系
	 chkconfig --add php-fpm
	 chkconfig php-fpm on
	
	 service php-fpm start # 启动 php-fpm
	 
	 # 重启 nginx
	 /usr/local/nginx/sbin/nginx -s stop  # 关闭
	 /usr/local/nginx/sbin/nginx # 开启 
	
	 netstat -tunpl | grep :80 # 查看 Nginx 端口 
	 
	```  
	  
	3.6) 测试 PHP 
	 
	```bash
	 # 编写 PHP 文件 phpinfo.php  
	 vi /usr/local/nginx/html/phpinfo.php
	 <?php 
	 	phpinfo(); 
	 ?>
	 
	```   
	  
	在浏览器中输入 http://192.168.1.128/phpinfo.php 成功 如下图：  
	![PHP info](http://p704q21lb.bkt.clouddn.com/2018-04-15-php-success.png)     

### 扩展篇  
1. 安装 memcache 扩展  
	1.1) 安装  
	  
	```bash
	 cd /lnmp/memcache-3.0.8
	 /usr/local/php/bin/phpize # 生成 .configure 编译文件
	
	 ./configure --with-php-config=/usr/local/php/bin/php-config \
	 --with-zlib-dir --enable-memcache
	
	 make && make install
	 
	```  
	
	1.2) 修改PHP配置文件 php.ini  
	  
	```bash
	 vi /usr/local/php/etc/php.ini
	 
     # 在891行下面添加 memcache.so 
     extension=memcache.so
     
     # 保存退出，重启 nginx
     /usr/local/nginx/sbin/nginx -s stop  # 关闭
	 /usr/local/nginx/sbin/nginx # 开启
	 
	```  
	  
	在浏览器中输入 http://192.168.1.128/phpinfo.php 查看memcache扩展是否安装成功 如下图：  
	![memcache success](http://p704q21lb.bkt.clouddn.com/2018-04-15-memcache-success.png)
	
	
2. 安装 memcache 服务器  
	2.1) 创建memcache用户,用于执行memcached进程 
	 
	```bash 
	 groupadd memcache # 创建memcache组
	 useradd -r -g memcache memcache # 创建memcache用户
	```  
	
	2.2) 安装 
	 
	```bash
	 # 安装 libevent
	 cd /lnmp/libevent-1.3
	
	 ./configure 
	
	 make && make install
	
	 # 生成链接
	 ln -s /usr/local/lib/libevent-1.3.so.1 /usr/lib/libevent-1.3.so.1 
	 ln -s /usr/local/lib/libevent-1.3.so.1 /usr/lib64/libevent-1.3.so.1
	
	 # 安装 memcached
	 cd /lnmp/memcached-1.4.31
	
	 ./configure --prefix=/usr/local/memcache --with-libevent=/usr/lib
	
	 make && make install
	
	 # 启动 memcache
	 /usr/local/memcache/bin/memcached -d -m 10 -u memcache -l 192.168.1.128 -p 11211 -c 128 -P/tmp/memcached.pid
	 
	 netstat -tunpl | grep :11211 # 查看 memcache 端口
	 
	```  
	2.3) 设置 memcache 开机启动 
	 
	```bash
	 # 编辑 /etc/rc.local 文件
	 vi /etc/rc.local
	 
	 # 在文件末行添加以下这行
	 /usr/local/memcache/bin/memcached -d -m 10 -u memcache -l 192.168.1.128 -p 11211 -c 128 -P/tmp/memcached.pid &> /dev/null	 
	```
	  

