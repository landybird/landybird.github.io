---
title: python pip 命令汇总
description: python pip 命令汇总
categories:
 - python
tags:
- python基础
---


# python pip 命令汇总

<br>


## 1 python 安装git上的package

>1 git clone 下载之后，进入文件目录， 执行 python setup.py  install

>2  pip install git+git://github.com/jkbr/httpie.git

>或者   pip install git+https://github.com/jkbr/httpie.git

<br>

## 2 列出已安装的包

    pip freeze or pip list

<br>

## 3 导出requirements.txt

    pip freeze  > <目录>/requirements.txt

<br>

## 4 卸载包，升级包

    pip uninstall <包名> 

    pip install -U <包名>

<br>

## 5 显示包所在的目录，搜索包


    pip show -f <包名>
    
    
    pip search <搜索关键字>
    


<br>

## 6 使用镜像源

    pip install <包名> -i https://mirrors.aliyun.com/pypi/simple

    全局配置镜像源
    
    $HOME/.pip/pip.conf文件中
    
    [global]
    timeout = 6000
      index-url = https://mirrors.aliyun.com/pypi/simple




<br>

## 7 python 指定 pip 安装目录

     pip install --target=/home/xx/xxxx/ virtualenvwrapper
     
    全局配置安装目录
    
    $HOME/.pip/pip.conf文件中
    
    [install]
    install-option=--prefix=~/.local

    然后再pip install virtualenvwrapper
    
    



<br>

## 8 python pip install --user  安装到自己的.local目录中

    pip install --user paramiko



    #  python  -m site --user-base -->>  查找自己的用户基础目录 /home/jiamin/.local
    
    #  vim ~/.basr_profile   
    	
	PATH=$PATH:$HOME/bin:$HOME/.local/bin

        source ~/.bash_profile

	设置环境变量，安装到user基础目录下的python模块可以全局使用


<br>

## 9 pip search <package_name> `查看pypi中所有存在的包`











