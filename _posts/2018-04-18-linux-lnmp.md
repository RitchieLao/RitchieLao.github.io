---
layout: post
title: '[LNMP] CentOS 7.2 搭建LNMP环境'
subtitle: 'CentOS 7.2 + Nginx 1.12.2 + MYSQL 5.7.16 + PHP 7.12'
date: 2018-04-18
categories: Linux
cover: ''
tags: WEB服务器
---

> LNMP 即：Linux、Nginx、MySQL和PHP的缩写，是当下最流行的WEB服务器架构之一。本文将带领你在CentOS 6.8系统下源码编译一套LNMP环境：    
> 环境参数：  
> 虚拟机：VMware Wokstation 14   
> 系统：CentOS 7.2  
> [ 本教程适用于 CentOS 7.x ]  


### 准备篇
在安装之前，首先要对服务器进行如下设置：  
  
1. 关闭 SELinux、防火墙  
	1.1) 关闭 SELiunx

	```bash  
	 # 编辑 SElinux 配置文件
	 vi /etc/selinux/config
	 SELINUX=disabled # 将此处改为 'disabled'
	 # SELINUXTYPE=targeted # 将此行注释掉

	 :x # 保存退出
	 setenforce 0 # 使配置立即生效
	 
	```

	1.2) 关闭 firewall

	```bash 
	 # 停止 firewall 
	 systemctl stop firewalld.service 

	 # 禁止 firewall 开机启动
	 systemctl disable firewalld.service 

	```

	1.3) iptables 设置

	```bash
	 # 安装 iptables
	 yum install iptables-services
	 
	 # 编辑 iptables 防火墙配置文件，在倒数第三行处添加以下两行
	 vi /etc/sysconfig/iptables
 
	 # 允许所有网段访问80、3306端口
	 -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
	 -A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT

	 :x # 保存退出

	 systemctl restart iptables.service # 重启 iptables
	 systemctl enable iptables.service #设置 iptables 开机启动
	 
	```   
 
2. 卸载系统自带的 php、httpd、mysql(或mariadb)，如能查询出相应的rmp包，则将其删掉
	2.1) 卸载 mariadb  
	
	```bash
	 # 查询 mariadb 
	 rpm -qa | grep mariadb
	 mariadb-libs-5.5.52-1.el7.x86_64
	
	 #删除 mariadb
	 rpm -e mariadb-libs-5.5.52-1.el7.x86_64 --nodeps
	 
	```  
	**注：**重复以上两步，把httpd和php也干掉   

3. 将虚拟机接入真实的网络  
	3.1) 把虚拟机的网卡设置为"桥接模式"

	3.2) 修改 IP 地址、添加 DNS  
	  
	```bash
	 # 编辑网络配置文件 ifcfg-eth0  
	 vi /etc/sysconfig/network-scripts/ifcfg-eth0

	 # 修改 IP 地址 
	 IPADDR=192.168.1.68
	 PREFIX=24
	 GATEWAY=192.168.1.1

	 # 添加 DNS
	 DNS1=8.8.8.8
	 DNS2=114.114.114.114

	 # 重启网络
	 service network restart
	 
	```  
	   
	**注：** IP地址、DNS，请根据自己的实际环境修改   
	 
4. 用 yum 方式安装编译工具及 LNMMP 环境所需的依赖包    
	  
	```bash  
	 # 安装编译工具
	 yum install -y gcc gcc-c++ make unzip prel wget
	 
	 # 安装所需的依赖包
	 yum install -y zlib-devel apr* autoconf automake bison bzip2 bzip2* \
cpp cloog-ppl compat* curl curl-devel t1lib t1lib* libxslt-devel* \
freetype freetype* freetype-devel gtk+-devel gd gettext ppl glibc \
gettext-devel libcom_err-devel libpng libpng* libpng-devel libjpeg* \
libsepol-devel libselinux-devel libstdc++-devel libtool* libgomp \
libxml2 libxml2-devel libXpm* libX* libtiff libtiff* mpfr nasm nasm* \
tp openssl-devel patch pcre-devel php-common php-gd policycoreutils \
fontconfig fontconfig-devel  
	
	```  
	   
5. 搭建 LNMP 环境所需的源码包  
	php-7.1.2.tar.gz         pcre-8.40.tar.gz          imagick-3.4.3.tgz  
	tiff-4.0.7.tar.gz        yasm-1.3.0.tar.gz         zlib-1.2.8.tar.gz  
	cmake-3.7.2.tar.gz       jpegsrc.v9b.tar.gz        libgd-2.2.0.tar.gz  
	t1lib-5.1.2.tar.gz       memcache-3.0.8.tgz        boost_1_59_0.tar.gz  
	mcrypt-2.6.8.tar.gz      mysql-5.7.16.tar.gz       nginx-1.12.2.tar.gz    
	freetype-2.7.1.tar.gz    mhash-0.9.9.9.tar.gz     libpng-1.6.28.tar.gz  
	openssl-1.1.0e.tar.gz  libmcrypt-2.5.8.tar.gz  memcached-1.4.35.tar.gz  
	pecl-memcache-php7.zip  ImageMagick-7.0.4-4.tar.gz 
	    
	**下载地址：** [源码包下载](https://pan.baidu.com/s/1CcqX0USu7rsBhPS-ioX2xQ) 密码： idx0 

6. 目录规划及源码包上传、解压  
	6.1) 目录规划  
		 源码包目录：/usr/local/src/lnmp  
		 源码包安装位置：/usr/local/源码包名称 (前半部分)

	6.2) 用 WinSCP 将源码包上传至指定目录下，编写 unpack.sh，批量解压源码包 
	  
	```bash
	 cd /usr/local/src/lnmp

	 vi unpack.sh
	 #!/bin/bash
	 # unpack.sh
	 # 解压各种格式的源码包脚本
	
	 cd /usr/local/src/lnmp
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

	 :x 保存退出
	
	 chmod a+x ./unpack.sh # 给 unpack.sh 执行权限 
	 ./unpack.sh # 运行脚本，解压源码包
	
	```

### 安装篇
1. 安装 Nginx  
	1.1) 创建 nginx 用户及用户组   
	 
	```bash
	 groupadd nginx # 创建 nginx 组  
	 
	 # 创建用户 nginx 并加入到 nginx 组，不允许其登录系统
	 useradd -g nginx nginx -s /bin/false 
	 
	```    
	
	1.2) 安装依赖包   
	 
	```bash
	 # 安装 pcre
	 cd /usr/local/src/lnmp/pcre-8.40
	 ./configure --prefix=/usr/local/pcre  
	 
	 make && make install
	
	 # 安装 openssl
	 cd /usr/local/src/lnmp/openssl-1.1.0e 
	 ./config --prefix=/usr/local/openssl  
	 
	 make && make install
	 
	 # 编辑 profile 文件，将 openssl 加入系统环境变量 
	 vi /etc/profile
	 
	 # 在文件末行添加以下这行
	 export PATH=$PATH:/usr/local/openssl/bin

	 :x # 保存退出
	 source /etc/profile # 重载，使之生效

	 # 安装 zlib
	 cd /usr/local/src/lnmp/zlib-1.2.8
	 ./configure --prefix=/usr/local/zlib  
	 
	 make && make install
	 
	```   
	 
	1.3) 安装 Nginx  
	  
	```bash
	 cd /usr/local/src/lnmp/nginx-1.12.2
	 ./configure \
	 --prefix=/usr/local/nginx \
	 --without-http_memcached_module \
	 --user=nginx --group=nginx \
	 --with-http_stub_status_module \
	 --with-http_ssl_module \
	 --with-http_gzip_static_module \
	 --with-openssl=/usr/local/src/lnmp/openssl-1.1.0e \
	 --with-zlib=/usr/local/src/lnmp/zlib-1.2.8 \
	 --with-pcre=/usr/local/src/lnmp/pcre-8.40
	
	 make && make install
	
	```   

	 **注：**openssl 、 zlib 、 pcre 扩展模块指向源码包的路径  
	   
	1.4) 编辑 nginx 配置文件 nginx.conf  
	  
	```bash  
	vi /usr/local/nginx/conf/nginx.conf  
	user nginx nginx; # 修改nginx的执行用户与组为 'nginx'	
	error_log logs/error.log; # 指定错误日志的保存路径
	
	pid /var/log/nginx/nginx.pid; # 指定 nginx.pid 的路径	
	
	worker_rlimit_nofile 65535; # 修改打开文件最大数为 65535	
	events {
		use epoll;
		worker_connections 65535; # 修改最大连接数为 65535
	}
	
	server {
		listen 80;
		server_name localhost;
	
		access_log  logs/host.access.log  main; # 开启日志记录

		location / {
			root html;

			index index.html index.htm;
		}
	}
	
	```   
	1.5) 启动&测试   
	  
	```bash   
	　/usr/local/nginx/sbin/nginx # 启动 Nginx  
	　  
	　netstat -tunpl | grep :80 # 查看 Nginx 端口  
	　pstree | grep nginx # 查看 Nginx 服务进程 
	　  
	```  
	
	在浏览器中输入 http://192.168.1.68 成功如下图：  
	  
	![Nginx success](http://p704q21lb.bkt.clouddn.com/2018-04-18-lnmp-nginx.png)   
	 
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

	 # 创建用户 mysql 并加入到 mysql 组，不允许其登录系统
	 useradd -g mysql mysql -s /bin/false 

	 
	 # 创建 mysql 安装目录及数据库存放目录
	 mkdir -p /usr/local/mysql/data

	 # ACL授权，设置 mysql 用户对数据库存放目录的权限
	 setfacl -m u:mysql:rwx -R /usr/local/mysql/data 
	 setfacl -m d:u:mysql:rwx -R /usr/local/mysql/data
	 
	```  
	2.2) 安装依赖包  
	  
	```bash 
	 # 安装 cmake
	 cd /usr/local/src/lnmp/cmake-3.7.2
	 ./configure  
	 
	 make && make install

	 # boost  
	 cp /usr/local/src/lnmp/boost_1_59_0 /usr/local/boost # 复制

	 # 安装 ncurses
	 yum install -y ncurses-devel
	 
	```  
	 
	2.3) 安装 MySQL   
	 
	```bash 
	 cd /usr/local/src/lnmp/mysql-5.7.16
	 cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
	 -DMYSQL_DATADIR=/usr/local/mysql/data/\
	 -DDOWNLOAD_BOOST=1 \
	 -DWITH_BOOST=/usr/local/boost \
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
	
	**注：**MySQL 5.7 以上的版本，安装编译时间有点长，大约需要40～60分钟左右  

	2.4) 生成 MySQL 配置文件 
	  
	```bash
	 # 将复制 MySQL 的默认配置文件到 /etc 目录下，并重命名为 my.cnf
	 cp /usr/local/mysql/support-files/my-default.cnf /etc/my.cnf 
	 
	 # 修改 /etc/my.cnf 配置文件
	 vi /etc/my.cnf
	 
	 # 在 [mysqld] 配置项下，修改 mysql 的安装目录和数据库存放目录
	 [mysqld]
	 basedir = /usr/local/mysql
	 datadir = /usr/local/mysql/data
	 
	```  
	2.5) 初始化 MySQL 数据库
	  
	```bash
	 /usr/local/mysql/bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data/ 

	```

	**注：**--initialize 生成默认密码   --initialize-insecure 不生成默认密码，密码为空 
	    
	2.6) 设置 MySQL 开机启动  
	  
	```bash
	 # 在安装目录的 support-files 目录下找到启动脚本 mysql.server，
	 # 将其复制到 /etc/init.d 目录下，并改名为 mysqld
	 cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld

	 # 编辑 mysqld
	 vi /etc/init.d/mysqld
	 basedir = /usr/local/mysql # 添加 mysql 的安装目录
	 datadir = /usr/local/mysql/data # 添加数据库存放目录

	 :x # 保存退出
	
	 # 给 mysqld 执行权限
	 chmod a+x /etc/init.d/mysqld
	
	 # 将 mysqld 纳入 chkconfig 管理体系
	 chkconfig --add mysqld
	
	 service mysqld start # 启动 mysql 
	
	 netstat -tunpl | grep :3306 # 查看 mysql 端口
	 pstree | grep mysqld # 查看 mysql 进程
	 
	```

	2.7) 将 mysql 添加到系统环境变量

	```bash
	 vi /etc/profile # 编辑 profile

	 # 在文件末行添加这行
	 export PATH=$PATH:/usr/local/mysql/bin

	 :x # 保存退出

	 source /etc/profile # 重载 profile 使之生效

	```  
	  
	2.8) 重置 root 密码   
	  
	```bash
	 mysql -uroot -p # 登录 mysql 
	
	 # 设置密码
	 update mysql.user set authentication_string=password('root018') where user='root';
	
	 # 更新数据库，使密码生效
	 flush privileges;
	 
	``` 

	2.9) 将 mysql 库文件添加到系统默认位置

	```bash
	 ln -s /usr/local/mysql/lib/mysql /usr/lib/mysql
	 ln -s /usr/local/mysql/include/mysql /usr/include/mysql

	```
 
3. 安装 PHP  
	3.1) 安装依赖包  
	  
	```bash  
	 # 安装 zlib
	 cd /usr/local/src/lnmp/zlib-1.2.8
	 ./configure 
	 
	 make && make install

	 # 安装 libmcrypt
	 cd /usr/local/src/lnmp/libmcrypt-2.5.8
	 ./configure --prefix=/usr/local/libmcrypt

	 make && make install

	 # 安装 libltdl
	 cd /usr/local/src/lnmp/libmcrypt-2.5.8/libltdl
	 ./configure --enable-ltdl-install

	 make && make install
	
	 # 设置环境变量
	 LD_LIBRARY_PATH=/usr/local/libmcrypt/lib:/usr/local/lib64 

	 # 安装 libpng
	 cd /usr/local/src/lnmp/libpng-1.6.28  
	 ./configure --prefix=/usr/local/libpng

	 make && make install
	 
	 # 安装 freetype
	 cd /usr/local/src/lnmp/freetype-2.7.1
	 ./configure --prefix=/usr/local/freetype

	 make && make install
	
	 # 安装 yasm
	 cd /usr/local/src/lnmp/yasm-1.3.0
	 ./configure 
	
	 make && make install
	 
	 # 安装 jpegsrc.v9b
	 cd /usr/local/src/lnmp/jpeg-9b
	 ./configure --prefix=/usr/local/jpeg \
	 --enable-shared \
	 --enable-static
	
	 make && make install
	
	 # 安装 tiff
	 cd /usr/local/src/lnmp/tiff-4.0.7
	 ./configure --prefix=/usr/local/tiff --enable-shared
	
	 make && make install
	 
	 # 安装 libgd
	 cd /usr/local/src/lnmp/libgd-2.2.0
	 ./configure --prefix=/usr/local/gd2 \
	 --with-jpeg=/usr/local/jpeg \
	 --with-freetype=/usr/local/freetype \
	 --with-fontconfig=/usr/local/freetype \
	 --with-xpm=/usr/lib64 \
	 --with-png=/usr/local/libpng \
	 --with-tiff=/usr/local/tiff \
	 --enable-shared
	
	 make && make install
	
	 # 安装 t1lib
	 cd /usr/local/src/lnmp/t1lib-5.1.2
	 ./configure --prefix=/usr/local/t1lib --enable-shared
	
	 make without_doc && make install

	 # 安装 mhash
	 cd /usr/local/src/lnmp/mhash-0.9.9.9
	 ./configure
	 
	 make && make install

	 # 添加软连
	 ln -s /usr/local/bin/libmcrypt-config /usr/bin/libmcrypt-config

	 # 添加环境变量
	 export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

	 # 安装 mcrypt
	 cd /usr/local/src/lnmp/mcrypt-2.6.8
	 ./configure --with-libmcrypt-prefix=/usr/local/libmcrypt

	 make && make install

	 # 安装 ImageMagick
	 cd /usr/local/src/lnmp/ImageMagick-7.0.4-4
	 ./configure --prefix=/usr/local/imagemagick/

	 make && make install  
	     
	 # 添加环境变量
	 export PKG_CONFIG_PATH=/usr/local/imagemagick/lib/pkgconfig
	 
	```   
	 
	3.2) 安装 PHP  
	  
	```bash 
	 # 若是64位系统，必须执行；否则安装php会出错
	 \cp -frp /usr/lib64/libltdl.so* /usr/lib/
	 \cp -frp /usr/lib64/libXpm.so* /usr/lib/

	 cd /usr/local/src/lnmp/php-7.1.2	 
	 ./configure --prefix=/usr/local/php \
	 --with-config-file-path=/usr/local/php/etc \
	 --with-mysqli=/usr/local/mysql/bin/mysql_config \
	 --with-mysql-sock=/tmp/mysql.sock \
	 --with-pdo-mysql=/usr/local/mysql \
	 --with-freetype-dir=/usr/local/freetype \
	 --with-gd=/usr/local/gd2 \
	 --with-png-dir=/usr/local/libpng \
	 --with-jpeg-dir=/usr/local/jpeg \
	 --with-openssl=/usr/local/openssl \
	 --with-xpm-dir=/usr/lib64 --with-mcrypt=/usr/local/libmcrypt \
	 --with-fpm-user=nginx --with-fpm-group=nginx --enable-fpm \
	 --with-xsl --with-zlib --enable-libxml --with-pcre-regex \
	 --enable-inline-optimization --enable-bcmath --enable-sockets \
	 --enable-gd-native-ttf --with-curl --with-gettext --with-iconv \
	 --with-pdo-sqlite --with-pear --enable-mbregex --enable-soap\
	 --enable-mbstring --enable-opcache --enable-pcntl --enable-shmop \
	 --enable-sysvsem --enable-xml --enable-zip --enable-mysqlnd
	
	 make && make install

	```  
	  
	3.3) 生成配置文件 
	 
	```bash
	 # 生成php配置文件 php.ini
	 cp /usr/local/src/lnmp/php-7.1.2/php.ini-production /usr/local/php/etc/php.ini
	
	 # 生成 php-fpm.conf 配置文件
	 cp /usr/local/php/etc/php-fpm.conf.default  /usr/local/php/etc/php-fpm.conf
	
	 # 生成www.conf配置文件
	 cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf
	 
	 # 编辑 php-fpm.con
	 vi /usr/local/php/etc/php-fpm.conf
	 pid = run/php-fpm.pid #去掉行首的';'
	 
	```   
	  
	3.4) 设置 PHP 开机启动  
	  
	```bash
	 # 在php源码目录中找到php的启动脚本init.d.php-fpm，将其复制到
	 # /etc/init.d 目录下，并改名为 php-fpm
	 cp /usr/local/src/lnmp/php-7.1.2/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
	
	 # 给 php-fpm 执行权限
	 chmod a+x /etc/init.d/php-fpm
	
	 # 将 php-fpm 纳入 chkconfig 管理体系
	 chkconfig --add php-fpm
	 chkconfig php-fpm on
	
	 service php-fpm start # 启动 php-fpm
	 
	 # 重载 nginx，使配置生效
	 /usr/local/nginx/sbin/nginx -s reload 
	
	 netstat -tunpl | grep :80 # 查看 Nginx 端口 
	 
	``` 

	3.6) 修改 nginx 配置文件，使其支持php

	```bash
	 # 编辑 nginx.conf
	 vi /usr/local/nginx/conf/nginx.conf

	 server {
		listen 80;
		server_name localhost;

		location / {
			root html;
	
			# 添加 index.php
			index index.html index.htm index.php;
		}
	
		# 添加php文件解析
		location ~ \.php$ {
			root html;

			fastcgi_pass 127.0.0.1:9000;
			fastcgi_index index.php;
			include fastcgi_params;
		}
	 }

	 :x # 保存退出

	 # 编辑文件 fastcgi_params 
	 vi /usr/local/nginx/conf/fastcgi_params
	
	 # 在第6行下添加这行
	 fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

	 :x # 保存退出

	 # 重载 nginx，使配置生效
	 /usr/local/nginx/sbin/nginx -s reload

	``` 
	  
	3.7) 测试 PHP 
	 
	```bash
	 # 编写 PHP 文件 phpinfo.php  
	 vi /usr/local/nginx/html/phpinfo.php
	 <?php 
	 	phpinfo(); 
	 ?>
	 
	```   
	  
	在浏览器中输入 http://192.168.1.128/phpinfo.php 成功 如下图：  
	  
	![PHP info](http://p704q21lb.bkt.clouddn.com/2018-04-18-lnmp-phpinfo.png)     

### 扩展篇 
1. PHP 相关设置  
	1.1) 将 php 添加到系统环境变量

	```bash
	 # 编辑 profile
	 vi /etc/profile 

	 # 在文末添加以下这行
	 export PATH=$PATH:/usr/local/php/bin

	 :x # 保存退出
	 
	 source /etc/profile # 重载 使之生效

	```

	1.2) 修改 PHP 配置文件 php.ini

	```bash
	 vi /usr/local/php/etc/php.ini

	 date.timezone = PRC # 将时区设置为 PRC

	 # 开启 opcache 缓存
	 [opcache]
	 zend_extension=opcache.so # 添加 opcache.so 扩展

	 # 把以下两行，行首的';'去掉
	 opcache.enable=1 
	 opcache.enable_cli=1

	 :x # 保存退出

	 # 重载 nginx，使配置生效
	 /usr/local/nginx/sbin/nginx -s reload

	```

2.  PHP扩展安装  
	2.1) 安装扩展  
	  
	```bash
	 # 安装 memcache
	 cd /usr/local/src/lnmp/pecl-memcache-php7
	 phpize # 生成 .configure
	
	 ./configure --with-php-config=/usr/local/php/bin/php-config \
	 --with-zlib-dir --enable-memcache
	
	 make && make install

	 # 安装 imagick
	 cd /usr/local/src/lnmp/imagick-3.4.3
	 phpize # 生成 .configure

	 ./configure --with-php-config=/usr/local/php/bin/php-config \
	 --with-imagick=/usr/local/imagemagick
	
	 make && make install

	 
	```  
	
	2.2) 修改 PHP 配置文件 php.ini  
	  
	```bash
	 vi /usr/local/php/etc/php.ini
	 # 'Dynamic Extensions' 配置项末尾
     # 添加 memcache.so、imagick.so 扩展
     extension=memcache.so
	 extension=imagick.so

     
     :x # 保存退出

	 service php-fpm reload # 重载 php

     # 重载 nginx，使配置生效
	 /usr/local/nginx/sbin/nginx -s reload
	 
	```   
	   
	2.3) 查看扩展是否安装成功  

	```bash   
	 # 查看 php 扩展目录中，是否有 memcache.so、imagick.so 模块
	 ls /usr/local/php/lib/php/extensions/no-debug-non-zts-20160303
	 imagick.so  memcache.so  opcache.a  opcache.so
	
	``` 
	  
	2.4) 在 phpinfo 页面中查看相关的扩展项 如下图   
	 
	imagick 扩展：
	
	![imagick success](http://p704q21lb.bkt.clouddn.com/2018-04-18-lnmp-php-imagick.png "imagick 扩展")  
	
	memcache 扩展：  
	
	![memcache success](http://p704q21lb.bkt.clouddn.com/2018-04-18-lnmp-php-memcache.png "memcache 扩展")
	
	
2. 安装 memcache 服务器  
	2.1) 创建 memcache 用户及用户组
	 
	```bash 
	 groupadd memcache # 创建 memcache 组
	
	 # 创建用户 memcache 并加入到 memcache 组，不允许其登录系统
	 useradd -g memcache memcache -s /bin/false

	```  
	
	2.2) 安装 
	 
	```bash
	 yum install -y libevent*

	 # 安装 memcached
	 cd /usr/local/src/lnmp/memcached-1.4.35
	 ./configure --prefix=/usr/local/memcache --with-libevent=/usr/lib
	
	 make && make install
	
	 # 启动 memcache
	 /usr/local/memcache/bin/memcached -d -m 10 -u memcache -l 192.168.1.68 -p 11211 -c 128 -P/tmp/memcached.pid
	 
	 netstat -tunpl | grep :11211 # 查看 memcache 端口
	 
	```  
	2.3) 设置 memcache 开机启动 
	 
	```bash
	 # 编辑 /etc/rc.d/rc.local 文件
	 vi /etc/rc.d/rc.local
	 
	 # 在文件末行添加以下这行
	 /usr/local/memcache/bin/memcached -d -m 10 -u memcache -l 192.168.1.68 -p 11211 -c 128 -P/tmp/memcached.pid &> /dev/null	 
	
	 :x # 保存退出

	 # 给 rc.local 执行权限
	 chmod a+x /etc/rc.d/rc.local

	```
