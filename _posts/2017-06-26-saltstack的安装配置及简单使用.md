---
title: saltstack的安装配置及简单使用
description: saltstack的安装配置及简单使用
categories:
- 自动化管理
tags:
- 自动化管理
---

<br>


# saltstack的安装配置及简单使用


**saltstack 和 Puppet Chef 一样可以让同时在多台服务器上执行命令,包括安装和配置软件。**


**Salt有两个主要的功能：`配置管理` 和 `远程执行`**



## 安装和配置



**如果用yum无法安装，提示：`No package salt-master available`.请先`安装epel源`**

```python
cd /usr/local/src/ 
wget http://mirrors.sohu.com/fedora-epel/6/x86_64/epel-release-6-8.noarch.rpm
rpm -ivh epel-release-6-8.noarch.rpm

然后再 yum 安装
```


- **安装，配置 master**

```python
master

	"""
	1. 安装salt-master
	yum install salt-master
	2. 修改配置文件：/etc/salt/master
	interface: 0.0.0.0    # 表示Master的IP 
	3. 启动
	service salt-master start
	"""
```
- **安装，配置 slave**

```python
slave

	"""
	1. 安装salt-minion
	    yum install salt-minion

	2. 修改配置文件 /etc/salt/minion
	    master: 10.211.55.4           # master的地址
	    或
	    master:
	        - 10.211.55.4
	        - 10.211.55.5
	    random_master: True

	    id: slave_id                   # 客户端在salt-master中显示的唯一ID
	3. 启动
	    service salt-minion start
	"""
```

- **授权**

```python
	"""
	salt-key -L                # 查看已授权和未授权的slave
	salt-key -a  salve_id      # 接受指定id的salve
	salt-key -r  salve_id      # 拒绝指定id的salve
	salt-key -d  salve_id      # 删除指定id的salve
	"""
```


- **执行命令**

```python
在master服务器上对salve进行远程操作

salt 'slave_id' cmd.run  'ifconfig'
```


- **py文件运行命令**

```python
pip install salt

import salt.client
local = salt.client.LocalClient()
result = local.cmd('c2.salt.com', 'cmd.run', ['ifconfig'])
```


- **设置开机启动**

```python
master主机执行：chkconfig salt-master on
minion主机执行：chkconfig salt-minion on
```


<br>

## saltstack的几个常用命令

<br>

### `state` 对被控制主机进行状态管理

state是`Saltstack最核心的功能`，通过预先定制好的sls（salt state file）文件`对被控制主机进行状态管理`，支持包括程序包（pkg）、文件（file）、网络配置（network）、系统服务（service）、系统用户（user）等。

- **(1) 将master上的一个文件，同步到客户端**

在 master 端 配置

```python
	1 cd /srv/salt

	2 mkdir mycode
		mycode/
			├── files
			│   └── xx.py
			└── init.sls

	3 vim init.sls 
			/data/xx.py: ID
			    file:
				- managed
				- source: salt://mycode/files/xx.py
				- user: root
				- makedirs: True
				- mode: 644
```

执行命令推送文件 

```python
salt '*' state.sls mycode
```

显示结果

```python
----------
ID: /data/xx.py
Function: file.managed
Result: True
Comment: File /data/xx.py updated
Started: 21:52:21.507086
Duration: 308.826 ms
Changes:   
----------
diff:
  New file
mode:
  0644

Summary
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
```

<br>

- **(2) 将master上的文件夹，同步到客户端**


在master 配置

```python
	1 - cd /srv/salt

	2 - mkdir s7code
		s7code/
		├── files
		│   └── video
		│       ├── a.py
		│       ├── b.py
		│       └── c.py
		└── init.sls

	3 - vim init.sls 

		/data/xx.py:         # 对应的ID
		    file.recurse:    # 上传文件夹
			- name: /data/codes   # 上传的路径
			- source: salt://s7code/files/video   # 源路径
			- user: root
			- makedirs: True
			- file_mode: 644
			- dir_mode: 755
```

执行命令推送文件 

```python
salt '*' state.sls s7code
```


显示结果

```python
----------
ID: /data/xx.py
Function: file.recurse
Name: /data/codes
Result: True
Comment: Recursively updated /data/codes
Started: 22:06:00.073110
Duration: 417.839 ms
Changes:   
----------
/data/codes/1.py:
  ----------
  diff:
      New file
  mode:
      0644
/data/codes/2.py:
  ----------
  diff:
      New file
  mode:
      0644
/data/codes/3.py:
  ----------
  diff:
      New file
  mode:
      0644

Summary
------------
Succeeded: 1 (changed=1)
Failed:    0
------------
Total states run:     1
```

<br>

### `grains`  用于保存服务器的静态信息


**内置grains**

	- salt "slave1" grains.ls
	- salt "slave2" grains.items
	- salt "*" grains.get key 


**自定义grains**

```python
	- minion 
		vim /etc/salt/minion 
			grains:
			  roles:
				- webserver
				- memcache
			  deployment: datacenter4
			  cabinet: 13
			  cab_u: 14-15

		在master上：
			salt '*' saltutil.sync_grains
			salt '*' grains.get cabinet

	- master 
		vim /srv/salt/_grains/xx.py

			import time
			def xxo():
				s = {}
				s['xkkkkkkkkkkkkk1'] = str(time.time())
				s['xkkkkkkkkkkkkk2'] = 123
				return s

		salt 'w1.com' saltutil.sync_grains
```		

**应用： 在state中根据目标服务器的信息不同，做不同处理。**


<br>

### `pillar` 用于保存服务器的动态信息


**内置pillar**

    vim /etc/salt/master
	pillar_opts: True

    PS: 重启生效
    
    
**自定义pillar**

```python
       vim /srv/pillar/top.sls
                base:
                  'c*':
                    - apache

            vim /srv/pillar/apache.sls
                x1:
                { % if grains['os_family'] == 'Debian' % }
                  apache: apache_1
                  { % elif grains['os_family'] == 'RedHat' % }
                  apache: httpd_2
                  { % elif grains['os'] == 'Arch' % }
                  apache: apache_3
                { % endif % }
                x2:
                { % if grains['ip_interfaces'].get('eth0')[0].startswith('10.10') % }
                  nameservers: ['10.10.9.31','10.10.9.135']
                  zabbixserver: ['10.10.9.234']
                { % else % }
                  nameservers: ['10.20.9.75']
                  zabbixserver: ['10.20.9.234']
                { % endif % }


            PS: 刷新 salt '*' saltutil.refresh_pillar
```

**自定义ext_pillar**

```python
	vim /etc/salt/master
                ext_pillar:
                    - wupeiqi:
                    或
                    - wupeiqi:{api:http://www.oldbody.com}
            vim /usr/lib/python2.7/site-packages/salt/pillar/wupeiqi.py
                import time
                import commands
                import salt.client

                def ext_pillar(minion_id,pillar,*args,**kwargs):

                    local = salt.client.LocalClient()
                    result = local.cmd(minion_id, 'cmd.run', ['ifconfig'])
                    return {'ifconfig':result.get(minion_id)}

            PS: 修改配置文件后，需要重启
```

<br>

### 代码发布简单流程

```python
	1. 在master上拉代码
	    如果文件存在，pull；否则clone
	    
	    subprocess.check_call('git clone https://gitee.com/melo/video.git', cwd='/Users/landybird/PycharmProjects/demo', shell=True)

	2. build
	    使用编译器进行编译

	3. 软连接到saltstack

	    ln -s /data/codes/video /srv/salt/mycode/files/video
	    
	3. 将代码推送到指定机器
	
	    xxxxxx:
	      file.recurse:
		- name: /data/codes
		- source: salt://mycode/files/video
		- user: root
		- makedirs: True
		- file_mode: 644
		- dir_mode: 755

	    salt 'c1.com' state.sls mycode

	4. 将代码推送到所有机器
	
	    salt '*' state.sls mycode
```	    
