---
layout: post
title: '[Mac OS] Mac OS 安装 pyenv 管理多版本 Python'
subtitle: ''
date: 2018-04-22
categories: Mac OS
cover: ''
tags: 操作系统
---

> pyenv 是一个 Python 版本管理工具，它能够进行全局的 Python 版本切换，也可以为单个项目提供对应的 Python 版本。使用 pyenv 以后，可以在 Mac OS 上安装多个不同的 Python 版本。

1. 安装 pyenv  

	```bash  
	 # 使用 homebrew 安装 pyenv
	 brew install pyenv

	 # 编辑当前用户 home 目录下的 .bash_profile 文件
	 cd 
	 vi .bash_profile

	 # 在文件的末尾添加以下两行
	 export PYENV_ROOT=/usr/local/var/pyenv
	 if which pyenv > /dev/null; then eval "$(pyenv init -)"; fi

	 :x # 保存退出

	 source .bash_profile # 重载，使配置生效

	```

2. 使用 pyenv 安装 Python (以 Python 3.6.3 为例)   

	```bash
	 # 安装 Python 3.6.3
	 pyenv install 3.6.3 

	 pyenv rehash # 更新版本库

	 # 设置系统的默认Python版本为 3.6.3
	 pyenv global 3.6.3

	```

3. pyenv 的常用命令  

	```bash
	 # 查看当前已安装的 Python 版本
	 pyenv versions 

	 # 查看可安装的 Python 版本
	 pyenv install --list 

	 # 安装一个指定的 Python 版本 如 3.6.3
	 pyenv install 3.6.3

	 # 更新 pyenv 版本库
	 pyenv rehash

	 # 设置全局 Python 版本 如 3.6.3
	 pyenv global 3.6.3

	 # 查看当前使用的 Python 版本
	 pyenv version

	 # 设置局部 Python 版本 如 3.6.3
	 pyenv local 3.6.3

	 # 运行临时的 Python 版本 如 3.6.3
	 pyenv shell 3.6.3 

	 # 卸载一个指定的 Python 版本 如 3.6.3
	 pyenv uninstall 3.6.3 

	```  

