---
layout: post
title: '[Mac OS] Mac OS 下 pipenv 的安装与配置'
subtitle: ''
date: 2018-04-22
categories: Mac OS
cover: ''
tags: 操作系统
---

> pipenv 整合了 virtualenv, pip, pipfile, 用于更方便地为项目建立虚拟环境并管理虚拟环境中的第三方模块.。

1. 安装 pipenv  

	```bash  
	 # 安装
	 pip install pipenv
	 
	```

2. 新建虚拟环境   

	```bash
	 mkdir /python/fisher # 新建目录

	 cd /python/fisher # 进入目录

	 # 创建 Python 解释器，如不指定 python 的版本，
	 # 则会使用系统默认的 python 版本
	 pipenv install
	
	 pipenv shell # 运行虚拟环境
	 
	```

3. pipenv 的常用命令  

	```bash
	 # 创建 Python 解释器
	 pipenv install

	 # 运行虚拟环境
	 pipenv shell

	 # 显示当前目录路径
	 pipenv --where 

	 # 查看环境信息
	 pipenv --venv

	 # 查看 Python 解释器
	 pipenv --py

	 # 为虚拟环境安装相关的模块 如 requests
	 pipenv install requests

	 # 查看当前环境安装的库及其依赖
	 pipenv graph

	 # 检查安全漏洞
	 pipenv check

	 # 卸载全部包并从Pipfile中移除
	 pipenv uninstall --all 

	```  

