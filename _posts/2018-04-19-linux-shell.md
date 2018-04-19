---
layout: post
title: '为 NGINX 写一个启动脚本 nginxd'
subtitle: ''
date: 2018-04-19
categories: linux
cover: ''
tags: Linux
---

> Linux系统中，服务脚本都放在 /etc/init.d 目录下；由于通过源码安装的Nginx，没有自带启动脚本；管理起来较为繁琐；因此，我们可以参照标准的服务脚本来自定义一个 nginxd 启动脚本

1. 通过 cat 查看 /etc/init.d 目录下的服务脚本，我们发现：在每个脚本头部都有以下几行：  
	\#  
	\# description: xxx   
	\#  
	\# chkconfig: 2345 x x    
	\# description: xxx  
	\#  
	\# processname: xxxx  
	 
2. 动手写一个 nginxd  
   
	```bash
	 vi /etc/init.d/nginxd  
	 #!/bin/bash  
	 #  
	 # description: 启动 Nginx 服务器 
	 #
	 # chkconfig: 2345 90 20  
	 # description: Nginx server daemon  
	 #  
	 # processname: nginxd  
	 #   
	  	
	 # 启动  
	 run() {  
	   /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf  
	   sleep 1  
		
	   if [[ $? == 0 ]]; then  
	     echo "启动 Nginx: [OK]"  
	   else
	     echo "启动 Nginx: [FAILED]"
	   fi	
	 }
		
	 # 停止
	 shtudown() {
	   /usr/local/nginx/sbin/nginx -s stop
	   sleep 1
		
	   if [[ $? == 0 ]]; then
	     echo "关闭 Nginx: [OK]"
	   else
	     echo "关闭 Nginx: [FAILED]"
	   fi
	 }
		
	 # 重启
	 reboot() {
	   /usr/local/nginx/sbin/nginx -s stop
	   echo "关闭 Nginx: [OK]"
	   sleep 1
		
	   if [[ $? == 0 ]]; then
	     /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
	     echo "启动 Nginx: [OK]"
	   else
	     echo "重启 Nginx: [FAILED]"
	   fi	
	 }
		
	 # 重载配置
	 reload() {
	   /usr/local/nginx/sbin/nginx -s reload
	   sleep 1
		
	   if [[ $? == 0 ]]; then
	     echo "重新载入 nginxd: [OK]"
	   else
	     echo "重新载入 nginxd: [FAILED]"
	   fi
	 }
		
		
	 case $1 in
	   start )
	     run
	     ;;
		
	   restart )
	     reboot
	     ;;
		
	   stop )
	     shtudown
	     ;;
		
	   reload )
	     reload
	     ;;	
	 
	   *)
	    echo "可选项: {start|restart|stop|reload}"
	    ;;				
 	esac

	```   
	 
3. 给 nginxd 执行权限，并将其加入到 chkconfig 体系中  
   
	```bash  
	 chmod a+x /etc/init.d/nginxd # 赋予 x 执行权限
	 
	 chkconfig --add nginxd # 加入到 chkconfig
	 
	```   
	   
4. 完成以上2、3步，就可以使用用 "serivce nginxd doing" 方式，来管理 nginx 服务器 如  
  
	```bash  
	 service nginxd start # 启动 Nginx  
	```  	