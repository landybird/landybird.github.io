---
title: elastic search的简单使用
description: elastic search的简单使用
categories:
- elasticsearch
tags:
- elasticsearch
---

<br>



# elasticsearch简单认识


<br>

## elasticsearch的基本概念 

**ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。**

**Elasticsearch是用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。**


**特点：**

- **高效的搜索方案(搜索解决方案运行速度快)**
- **零配置,完全免费的搜索模式**
- **可以通过json和http与搜索引擎 交互（基于RESTfulweb接口）**
- **搜索服务器稳定**
- **扩展性好（扩展到数百台）**
- **基于 lucene(基于java开发) 的搜索服务器**




**应用：**

		用于分布式全文检索
	
		ELK 日志分析系统 


<br>

## 各类数据库的搜索对比



### 关系型数据库


**关系型数据的like,和正则regex 可以满足简单的搜索功能,但是：**

- **1 无法打分**
- **2 没有分布式**
- **3 没法进行搜索解析 (关键词，分词)**
- **4 效率低 (数据量大)**
- **5 无法分词**


### elasticsearch可以理解为一种 `nosql-文档型的非关系型数据库`

**json (数据的多表关系)，直接保存成 json 文档，完成数据储存 + 数据的搜索**




<br>

## 安装 部署 



### 首先需要安装 java (windows 版本)

**下载地址：**


		http://www.oracle.com/technetwork/java/javase/
		downloads/jdk8-downloads-2133151.html



### 安装 elasticsearch-rtf(集成很多插件) 比原始版本更好

**下载** 


		git --- elasticsearch-rtf

		https://github.com/medcl/elasticsearch-rtf
		https://github.com/medcl/elasticsearch-rtf.git

**要求** 
 
		  JDK8+
		  x系统内存>2G

**运行**

	1 bin 下 shift + 右键  在命令行运行 

	>> elasticsearch.bat

	2 浏览器访问 127.0.0.1:9200

```python
	 	 {
	  "name" : "uabVDGq",
	  "cluster_name" : "elasticsearch",
	  "cluster_uuid" : "0skcOheDQQ2xuclrZGsUag",
	  "version" : {
	    "number" : "5.1.1",
	    "build_hash" : "5395e21",
	    "build_date" : "2016-12-06T12:36:15.409Z",
	    "build_snapshot" : false,
	    "lucene_version" : "6.3.0"
					  },
		  "tagline" : "You Know, for Search"
		}
```

### elasticsearch-head 插件 和 kibana



- **（1）elasticsearch-head**


**`浏览器样式的 elasticsearch 的操作工具` (类似于 mysql 的 navicat)**


**安装**
	   https://github.com/mobz/elasticsearch-head

**运行**  

```python
1 需要安装 node js (npm 命令)

	cd elasticesear-head
	cnpm === (npm的淘宝源镜像命令)  
	$ npm install -g cnpm --registry=https://registry.npm.taobao.org


	cnpm install  
	cnpm run start  # 有些包可能没有安装全，可以单独用 cnpm install packename

出现 Started connect web server on http://localhost:9100 标识成功


2 连接运行时候的 问题  === elasticsearch 的安全策略
	
	elasticsearch的 配置文件 
	
		http.cors.enabled: true
		http.cors.allow-origin: "*"
		# http:cors.allow-methods: OPTIONS,HEAD,GET,POST,PUT,DELETE
		# http:cors.allow-headers: "X-Requested-With,Content-Type,Content-Length,X-User"
```



- **(2) kibana 需要和 elasticsearch版本一致**

**Kibana是一个开源的分析与可视化平台，设计出来用于和Elasticsearch一起使用的。你可以用kibana搜索、查看、交互存放在Elasticsearch索引里的数据**

**运行启动：**

	kibana-5.1.2-windows-x86\bin  运行kibana.bat  连接

	监听 端口 5601
	 http://localhost:5601


	 elasticsearch 开启   9200
	 elasticsearch-head 开启 连接  9100
	 kibana 开启 连接  5601




# elasticsearch 的相关概念 


- **集群：  一个或者多个节点 组织在一起 (分布式的)**

- **节点： 一个节点是集群里的 一个 服务器，有一个名字来表示，默认是一个随机的 漫威漫画 角色名**

- **分片：将索引划分为 多分的能力，允许水平分割和扩展容量，多个分片响应请求，可以提高性能和吞吐量，将数据分成几份，放在不同的地方**

- **副本：创建分片的能力，复制数据(顶替的节点)**


<br>

**（1）与关系型数据库的对应关系(elasticsearch会维护自己的数据，进行索引)**

	elasticsearch  mysql

	index  ----  数据库

	type   -----  表

	document  ----  行

	fields  -----   列


**（2）HTTP 方法**

	http 1.0  get post head

	http 1.1 options put delete trace  connect


	GET  请求页面信息
	POST 添加数据
	PUT  传输数据
	DELETE  删除数据

	对应 RESTful api 接口



**（3）倒排索引**

**目前搜索引擎 的底层的 索引存储都是倒排索引**

**这是它区别于 关系型数据库和nosql的关键核心**

<br>


- **定义：根据属性值来确定记录的位置**

  	例如 三个 文件 A B C

		要查询里面 有 python 关键字的时候 

		正常情况：
			A B C 文件都要遍历,
		倒排索引:
			在数据存储的时候 进行分词,打分

			关键词              文章

			python            [文章1，<2,10>,2] [文章2,<12,23,4> 3]

			java              文章3 <2,4,5> 3

			redis             文章4，文章5



- **需要解决的问题：**


		1.大小写 转换的问题，python 与 PYTHON
		2.词干抽取，looking 和 look 应该 处理为一个词
		3. 分词，屏蔽系统 -- 屏蔽 系统
		4.倒排索引文件过大，压缩编码


<br>

# elasticsearch 的启动使用 


- **（一）依次启动**

		elasticsearch
		elasticsearch-head
		kibana


**(二) 索引初始化  -对应关系数据库的  数据库(创建数据库)**
					
```python
			# 创建索引  

				PUT lagou
				{
				  "settings": {
				    "index":{
				      "number_of_shards":5,
				      "number_of_replicas":1
				    }
				  }
				}

			# 获取 settings

				GET lagou/_settings

				GET _all/_settings

				GET .kibana,lagou/_settings

			#修改settings 

				PUT lagou/_settings
				{
				  "number_of_replicas": 2
				}


			# 获取索引信息
				GET _all
				GET lagou获取索引信息信息


		---------------------------------------------
			新建保存 文档  -  --增  PUT
		-----------------------------------------
		（1）	PUT lagou/job/1  --- 指明 id
			{
				"title":"python分布式爬虫",
				"salary_min":15000,
				"city":"北京",
				"company":{
					"name":"百度",
					"company_addr":"软件园"
				},
				"pulish_date":"2017-09-28",
				"comments":16
			}


		（2）	PUT lagou/job/  -- 不指明 id 自动生成
			{
			"title":"python web开发",
			"salary_min":18000,
			"city":"北京",
			"company":{
				"name":"美团",
				"company_addr":"软件园"
			},
			"pulish_date":"2017-09-28",
			"comments":16
			}

	------------------------------------------
		查看 文档记录          查    GET  ?_source=keyword
	------------------------------------------

		GET lagou/job/1   

		GET lagou/job/1?_source=title,city

		GET lagou/job/1?_source  


	------------------------------------------
		修改         PUT 覆盖 ,POST _update -- doc
	------------------------------------------

	PUT lagou/job/3
		{
		  "title":"python web开发",
		  "salary_min":18000,
		  "city":"北京",
		  "company":{
		  	"name":"美团",
		  	"company_addr":"软件园"
		  },
		  "pulish_date":"2017-09-28",
		  "comments":16
		} 




	POST lagou/job/3/_update
	{
	"doc":{
		"comments":50
		}
	}

	------------------------------------------
		删除        删除索引 ，删除记录 (不能删除文档，表)
	------------------------------------------

	DELETE lagou/job/3
	DELETE lagou
```


- **(三)  批量操作 (减少 http 三次握手)**



			------------------------------------------
					   批量查询
			------------------------------------------
		
		（1）	GET _mget
			{
			  "docs":[
			  {
			    "_index":"testdb",
			    "_type":"job",
			    "_id":1
			  },
			  
			  {
			    "_index":"testdb",
			    "_type":"job2",
			    "_id":1
			  }
			]
		}
		
		
		（2）GET testdb/_mget    #  index相同的情况下
		{
		  "docs":[
		  {
		    "_type":"job",
		    "_id":1
		  },
		  
		  {
		    "_type":"job2",
		    "_id":1
		  }
		]
		}
		
		（3）GET testdb/job2/_mget  # type 也相同
			{
			  "ids":[1,2,3]
			}
   




- **(四) bulk 批量 操作**


```python
	插入

		POST _bulk
		{"index":{"_index":"testdb","_type":"job2","_id":"4"}}
		{"title":"python web开发工程师","salary_min":"20000"}
		{"index":{"_index":"testdb","_type":"job","_id":"4"}}
		{"title":"python web开发工程师","salary_min":"10000"}

	删除

		POST _bulk
		{"delete":{"_index":"testdb","_type":"job2","_id":"4"}}
		{"delete":{"_index":"testdb","_type":"job","_id":"4"}}


	创建
		POST _bulk
		{"create":{"_index":"testdb","_type":"job2","_id":"4"}}
		{"title":"python web开发工程师","salary_min":"20000"}
		{"create":{"_index":"testdb","_type":"job","_id":"4"}}
		{"title":"python web开发工程师","salary_min":"10000"}

	更新
		POST _bulk
		{"update":{"_index":"testdb","_type":"job2","_id":"4"}}
		{"doc":{"salary_min":"20200"}}
		{"update":{"_index":"testdb","_type":"job","_id":"4"}}
		{"doc":{"salary_min":"30000"}}
```




- **（五）mapping 映射  == 让索引的建立更加细致和完善 (相当于数据表 定义数据类型)**


**创建索引 的时候  -- 预先定义字段的类型和属性,并告诉elasticsearch 如何索引数据**

	
```python
		===========    内置类型     ============
		
			string(text)
		
			keyword    不会分词  原词匹配
		
			数字
		
			日期类型
		
			bool
		
			binary
		
			复杂类型  object字典 nested数组
		
			geo  地理位置
		
			专业类型  ip competion
		
		
		 ========     可以添加的属性  =======
		
		  属性                     适合类型                         
		
		  store                     all
		
		  index                  string
		
		  null_value                  all
		
		  analyzer                 all     搜索分析器
		
		  include_in_all           all  默认全文搜索
		
		  format				   date
		
	
	例子 ：     
			
		
		PUT lagou
		{
		  "mappings":{
				"job":{
				"properties":{
					"title":{"type":"text"},
					"salary":{"type":"integer"},
					"city":{"type":"keyword"},
					"company":{
						"properties":{
						  "name":{"type":"text"},
						  "addr":{"type":"text"}
						  }
					    },
					"publish_date":{
					  "type":"date",
						"format":"yyyy-MM-dd"
						},
					"comments":{"type":"integer"}	
					}
				}
			}
		}
		
		
		PUT _bulk
		{"create":{"_index":"lagou","_type":"job","_id":2}}
		{"title":"爬虫工程师","salary":"12000","city":"北京","company":{"name":"baidu","addr":"BJ"},"publish_date":"2017-09-12","comments":"25"}
		{"create":{"_index":"lagou","_type":"job","_id":3}}
		{"title":"爬虫工程师","salary":"12000","city":"北京","company":{"name":"baidu","addr":"BJ"},"publish_date":"2017-09-12","comments":"25"}
		
		
		GET lagou/job/_mappingP
```


**mappings 创建好之后，  不可以修改类型**






- **（六）查询**


**elasticsearch 查询分类**

	基本查询  使用 elastisearch 内置的查询条件
	组合查询  把多个查询组合在一起进行 复合查询
	过滤     通过filter 在不影响打分的情况下筛选数据



**准备数据**

```python
		PUT searchtest
		{
		  "mappings":{
				"job":{
				"properties":{
					"title":{"type":"text","store": true,"analyzer": "ik_max_word"},
					"company_name":{"type":"keyword","store": true},
					"add_date":{
					  "type":"date",
						"format":"yyyy-MM-dd"
						},
					"comments":{"type":"integer"},
					"desc":{"type":"text"}
					}
				}
			}
		}


		PUT _bulk
		{"create":{"_index":"searchtest","_type":"job","_id":1}}
		{"title":"安卓工程师","company_name":"美团","add_date":"2017-09-12","comments":"25","desc":"美团网[1]，是2010年3月4日成立的团购网站。美团网有着“吃喝玩乐全都有”和“美团一次美一次”的服务宣传宗旨。总部位于北京市朝阳区望京东路6号。"}
		{"create":{"_index":"searchtest","_type":"job","_id":2}}
		{"title":"python工程师","company_name":"美团","add_date":"2014-09-12","comments":"2","desc":"美团网[1]，是2010年3月4日成立的团购网站。美团网有着“吃喝玩乐全都有”和“美团一次美一次”的服务宣传宗旨。总部位于北京市朝阳区望京东路6号。"}
		{"create":{"_index":"searchtest","_type":"job","_id":3}}
		{"title":"python webkaifa","company_name":"美团","add_date":"2015-08-12","comments":"22","desc":"美团网[1]，是2010年3月4日成立的团购网站。美团网有着“吃喝玩乐全都有”和“美团一次美一次”的服务宣传宗旨。总部位于北京市朝阳区望京东路6号。"}
		{"create":{"_index":"searchtest","_type":"job","_id":4}}
		{"title":"python爬虫工程师","company_name":"美团","add_date":"2016-04-12","comments":"12","desc":"美团网[1]，是2010年3月4日成立的团购网站。美团网有着“吃喝玩乐全都有”和“美团一次美一次”的服务宣传宗旨。总部位于北京市朝阳区望京东路6号。"}
```



**基本查询**

```python
	
	# match    模糊 分词()
	
					GET searchtest/_search
					{
					  "query": {
					    "match": {
					      "title": "爬"
					    }
					  }
					}
	
	# match_all {}  返回全部数据
	
	# match_phrase  match 词组 slop 表示 词语之间的 距离
	
				GET searchtest/_search
				{
				  "query": {
				    "match_phrase": {
				      "title": {"query":"python师",
				                "slop":6
				      }
				      
				    }
				  },
					}
	
	# multi_match   多个字段查询
	
						GET searchtest/_search
						{
						  "query": {
						    "multi_match": {
						      "query":"爬虫",
						      "fields": ["title^4","desc^3"]  # 权重
						    }
						  },
						  "from": 0,
						  "size": 4
						}
	
	
	
						"_score": 7.4276733,  表示命中的分数
	
	
	
	
	#  term  不分词 不会做解析
	
	# terms : [, , ,] 可以加一个数组
	
					GET searchtest/_search
					{
					  "query": {
					    "terms": {
					      "title": ["工程师","python"]
					    }
					  }
					}
	
	
	
	#  控制查询结果数量 from，size
	
					GET searchtest/_search
					{
					  "query": {
					    "match": {
					      "title": "python"
					    }
					  },
					  "from": 0,
					  "size": 4
					}
	
	
	
	#  只返回 指定的字段
	
	
			GET searchtest/_search
			{
			  "stored_fields": ["title","desc","company_name"], # 只返回stored 为 true的字段
			  "query": {
			    "multi_match": {
			      "query":"爬虫",
			      "fields": ["title^4","desc^3"]
			    }
			  },
			}
	
	
	#  sort 排序
	
	
				GET searchtest/_search
				{
				  "query": {
				    "match_all": {}
				  },
				  "sort": [
				    {
				      "comments": {
				        "order": "desc"  # asc
				      }
				    }
				  ]
				}
	
	
	#  查询范围
	
	
				（1） 整数
	
					GET searchtest/_search
					{
					  "query": {
					    "range": {
					      "comments": {
					        "gte": 10,
					        "lte": 20,
					        "boost": 2  # 权重
					      }
					    }
					  }
					}
	
	
					（2） 时间
	
					GET searchtest/_search
					{
					  "query": {
					    "range": {
					      "add_date": {
					        "gte": "2016-9-12",
					        "lte": "now",       # 小于当前时间
					        "boost": 2
					      }
					    }
					  }
					}
	

	
	#   模糊查询  wildcard  * 通配符
	
	
	
				GET searchtest/_search
				{
				  "query": {
				    "wildcard": {
				      "title": {
				        "value": "*python*",  # 模糊查询
				        "boost": 2
				      }
				    }
				  }
				}
	
	#  fuzzy 查询 --- 模糊匹配
	
	
	     ** 编辑距离 ：  两个字符串 之间相似程度的计算方法
	                   把一个字符串 变成另一个字符串 需要操作的最少次数
	
	               例子： ed("sail","sial")  --->> 1
	                     sailn  failing   --->>  3
	
	
	GET /_search
	{
	"query":{
	    "fuzzy":{
	        "title":{
	            "value":"linux",
	            "fuzziness":2,    # 编辑距离
	            "prefix_length":0 # 搜索词最前的位数
	            }
	        }
	    }
	}
	
	# suggest 搜索  search()
	
	GET jobbole/_search?pretty
	{
	"suggest":{
	    "my-suggest":{
	        "text":"linux",
	        "completion":{
	            "field":"suggest",
	            "fuzzy":{
	                "fuzziness":1, # 编辑距离
	                },
	            }
	        }
	    },
	    "_source":"title"
	}



**布尔查询  组合查询中 用的最多 bool**



	
	bool 
	
		filter   用来 筛选
	
		must
		should
		must_not  用来限定范围
	
	
	must + filter
	
	
						GET booltest/job/_search
						{
						  "query": {
						    "bool": {
								      "must": [
								        {
								          "match_all":{}
								        }
								      ],
	
								      "filter": {
								        "term": {
								          "salary": 20
								        }
								      }
						    }
						  }
						}
	
	
	
	
			GET booltest/job/_search
			{
			  "query": {
			    "bool": {
			      "must": [
			        {"match_all": {}}
			      ],
			      "filter": {
			        "terms": {
			          "title": [
			            "python",
			            "scrapy"
			          ]
			        }
			      }
			    }
			  }
			}
	
	
			select * from testjob  where (salary=20 OR title=python) AND (salary!=30)
	
						GET booltest/job/_search
						{
						  "query": {
						    "bool": {
						      "should": [
						        {"term":{"salary": 20}},
						        {"term":{"title":"python"}}
						      ],
						      "must_not": [
						        {"term":{"salary":30}}
						      ]
						    }
						  }
						}
	
	
	
	
	#   嵌套 查询 
	
	
						
				GET booltest/job/_search
				{
				  "query": {
				    "bool": {
				      "should": [
				        {"term":{"title":"python"}},
	
				        {"bool": {
				        	"must": [
	
					          {"term":{"title":"scrapy"}},
	
					          {"range": {
					            "salary": {
					              "gte": 10,
					              "lte": 30
				            }
				          }}
				        ]}}
				      ]
				    }
				  }
				}
	
	
	
	过滤 不存在的 field
	
			GET booltest/joba/_search
	
			{
			  "query": {
			    "bool": {
			      "filter": {
			        "exists": {
			          "field": "tags"
			        }
			      }
			    }
			  }
			}
	
	
	
	
	#    查看 分析器 解析 结果
	
	
			GET _analyze
			{
				"analyzer":"ik_max_word",
				"text":"python爬虫"   ---- >>>  python  爬 虫  爬虫
			}
	
	
	
	
			GET _analyze
			{
				"analyzer":"ik_smart",
				"text":"python爬虫"   ---->>>  python 爬虫
			}
```	
	


## 附：[基于Django + ES + Scrapy 实现简单的搜索引擎](https://github.com/melo108/doodlesearch)
































