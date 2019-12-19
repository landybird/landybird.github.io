---
title: linux 中安装 virtualenvwrapper
description: linux 中安装 virtualenvwrapper
categories:
 - python
tags:
- python基础
---


# linux 中安装 virtualenvwrapper(已经包含了virtualenv)

<br>


>1 创建 虚拟环境的目录

    $ export WORKON_HOME=~/Envs  临时设置
    $ mkdir -p $WORKON_HOME


>2  pip  install  virtualenvwrapper  --user （安装）`安装到.local中`


    # 可以设置  .bash_profile 中PATH = $HOME/jiamin/.local/bin 
                source ~/.bash_profile 使 virtualenvwrapper(virtualenv) 的相关命令 全局可用
                                                   

>3 设置VIRTUALENVWRAPPER_PYTHON

    vim ~/.bashrc
       
        export VIRTUALENVWRAPPER_PYTHON=/usr/local/bin/python3.6 # 指定python解释器的位置

	# 或者 mkvirtualenv -p /usr/local/bin/python3.6 venv_name
	       virtualenv -p /usr/local/bin/python3.5  venv_name
    
        source /home/jiamin/.local/bin/virtualenvwrapper.sh
    
    source ~/.bashrc
    
