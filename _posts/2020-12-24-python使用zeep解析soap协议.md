---
title: python使用zeep解析soap协议    
description: python使用zeep解析soap协议， zeep
categories:
- python
tags:
- python   
---

#### `REST`

The Representational State Transfer (REST)架构服务通过`统一资源定位器(URL)`公开。
  
    这个逻辑名称将资源的标识与所接受或返回的标识分开。

    要请求和检索资源，客户端将发出超文本传输协议(HTTP) GET请求。就是我们常见的post, get, put,delete,head等动作。


#### `SOAP`

基于` XML `的简易协议，是用`在分散或分布的环境中交换信息的简单的协议`，可使应用程序在 HTTP 之上进行信息交换。

    或者更简单地说：SOAP 是用于访问网络服务的协议
    
SOAP基于`XML语言`和`XSD标准`，其定义了一套编码规则，该规则定义如何将数据表示为消息，以及怎样通过HTTP协议来传输SOAP消息，它由以下四部分组成：

  SOAP信封（Envelope）：定义了一个框架，该框架描述了消息中的内容是什么，包括消息的内容、发送者、接收者、处理者以及如何处理这些消息。
  
  SOAP编码规则：它定义了一种系列化机制，用于交换应用程序所定义的数据类型的实例。
  
  SOAP RPC表示：它定义了用于表示远程过程调用和应答协定。
  
  SOAP绑定：它定义了一种使用底层传输协议来完成在节点间交换SOAP信封的约定。
  
SOAP消息基本上是从发送端到接收端的单向传输，它们常常结合起来执行类似于请求/应答的模式。
不需要吧SOAP消息绑定到特定的协议，SOAP可以运行在任何其他传输协议（HTTP、SMTP、FTP等）上。
另外，SOAP提供了标准的RPC方法来调用Web Service以请求/响应模式运行。


#### `zeep`的使用

- [Zeep: a fast and modern Python SOAP client](https://docs.python-zeep.org/en/master/)

demo

```python
import fire
import zeep
import json
from peewee import *

BASE_URL = "http://xxxxx:8088/services/getDeptInfoList"
BASE_URL_WSDL = "http://xxxxx:8088/services/getDeptInfoList?wsdl"
in0 = "ca50510c-c4c0-4c1f-ad15-7fd1a7a5ab671585296949599"
CONTENT_BODY = \
f"""
<soap:Envelope 
    xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" 
    xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <soap:Body>
        <ser:synDept>
            <ser:in0>{in0}</ser:in0>
        </ser:synDept>
    </soap:Body>
</soap:Envelope>
"""

MYSQL_CONF =  {
        'charset': 'utf8',
        'sql_mode': 'PIPES_AS_CONCAT',
        'use_unicode': True,
        'host': '10.0.0.209',
        'port': 3306,
        'user': 'name',
        'passwd': 'pwd'
        }

DB_NAME = "admin"
DATABASE = MySQLDatabase(DB_NAME, **MYSQL_CONF)


class BluefocusDepartment(Model):
    code = AutoField(column_name='CODE')
    departmentcode = CharField(column_name='DEPARTMENTCODE', index=True, null=True)
    name = CharField(column_name='NAME', null=True)
    parent_code = CharField(column_name='PARENT_CODE', null=True)
    parent_depcode = CharField(column_name='PARENT_DEPCODE', null=True)
    type = IntegerField(column_name='TYPE', null=True)
    create_time = DateTimeField(constraints=[SQL("DEFAULT CURRENT_TIMESTAMP")])
    d_status = IntegerField(constraints=[SQL("DEFAULT 0")], null=True)
    last_update = DateTimeField(constraints=[SQL("DEFAULT CURRENT_TIMESTAMP")], null=True)
    our_department_id = IntegerField(null=True)
    our_department_name = CharField(null=True)

    class Meta:
        database = DATABASE
        table_name = 'bluefocus_department'
        
        
class SyncCompanyDepartment(object):
    """
    syncAll： todo
    syncSpecifiedDepartment <department_code>  同步制定code
    all_dept   展示所有部门
    """

    def __init__(self):
        self._initial_all_department()

    def _initial_all_department(self):
        client = zeep.Client(wsdl=BASE_URL_WSDL)
        resp = json.loads(client.service.synDept(in0=in0))
        self.all_dept = resp.get("DeptInfoList", []) if isinstance(resp, dict) else []


    def syncAll(self):
            ...

    def syncSpecifiedDepartment(self, department_code):
        for i in self.all_dept:
            if department_code == i["DEPARTMENTCODE"]:
                code = i["ID"]
                departmentcode = department_code
                name = i["DEPARTMENTMARK"]
                parent_code = i["SUPDEPID"]
                parent_depcode = i["PARENTCODE"]

                try:
                    BluefocusDepartment\
                        .insert(code=code, departmentcode=departmentcode, name=name, parent_code=parent_code, parent_depcode=parent_depcode)\
                        .execute()

                    print(f"department_code: {department_code} saved!")

                except:
                    print(f"department_code: {department_code} existed!")




if __name__ == '__main__':
    fire.Fire(SyncCompanyDepartment)
       

```
