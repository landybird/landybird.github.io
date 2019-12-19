---
title: 使用 scrapy + mongoengine(ODM) 简单的爬取boss直聘上的工作信息
categories:
- 爬虫
tags:
- 爬虫
---

<br>

## 使用 scrapy + mongoengine(ODM) 简单的爬取boss直聘上的工作信息


<br>

**准备环境： virtualenv + scrapy + mongoengine, 安装下载即可**


<br>



### 1 `起始url分析`  


- 城市编码 -- 北京 c101010100

- 区域 -- 昌平区  'b_{0}-h_101010100'.format(quote('昌平区')), # 借助 urllib.request 的quote函数编码


- 过滤参数 -- params = dict(query='python开发', page=page) 


**最后拼接到一起即可**

```python
 url = 'https://www.zhipin.com/{0}/{1}/?{2}'.format(self.city_code, self.zone, urlencode(self.params))
```


<br>

### 2 `页码分析`, 每一页都可以得到 当前页的最大页码(通过选择器获取)，设置初始页面为 1 ，然后递加即可，直到加到等于最大页面值

<br>

### 3  `详细 detail_url 分析`, 通过选择器获取每一个工作的详细 url

<br>

### 4 `detail_parse` ,通过 itemloader 机制，进行预处理，除去空格，等等

<br>

### 5  `通过 mongoengine 定义模型类`，并结合item对象，`生成数据表和保存数据`

<br>


	
### 代码实现：	

#### boss_spider.py

```python
import scrapy
from scrapy import Request
from urllib.request import quote      # 转换单词为url编码
from urllib.parse import urlencode    # 转换 字典关键字为url编码

from ..items import BossjobItem,BossItemLoader  


class JobSpider(scrapy.Spider):
    name = 'job'
    allowed_domains = ['zhipin.com']
    start_urls = ['https://zhipin.com/']

    page = 1
    city_code = 'c101010100'
    zone = 'b_{0}-h_101010100'.format(quote('昌平区'))
    params = dict(query='python开发', page=page)

    def start_requests(self):
        url = 'https://www.zhipin.com/{0}/{1}/?{2}'.format(self.city_code, self.zone, urlencode(self.params))
        yield Request(url,dont_filter=False,callback=self.parse)

    def parse(self, response):
        last_page = response.xpath('//div[@class="page"]/a[last()-1]/text()').extract_first()
        if self.page < int(last_page):
            self.page +=1        # 控制页码
            url = 'https://www.zhipin.com/{0}/{1}/?{2}'.format(self.city_code, self.zone, urlencode(self.params))
            yield Request(url,dont_filter=False,callback=self.parse)

        job_base = 'https://www.zhipin.com{0}'
        job_urls = [job_base.format(item) for item in  response.xpath("//div[@class='info-primary']/h3/a/@href").extract()]
        for url in job_urls:
            yield Request(url,dont_filter=False,callback=self.detail_parse)


    def detail_parse(self,response):
        # 基于itemloader 实现 
        itemloader = BossItemLoader(item=BossjobItem(),response=response)
        itemloader.add_xpath('title',"//div[@class='info-primary']//h1/text()")
        itemloader.add_xpath('desc',"//div[contains=(@class='job-primary')]/div[@class='info-primary']/p/text()")
        itemloader.add_xpath('salary',"//div[@class='info-primary']//span[@class='badge']/text()")
        itemloader.add_xpath('job_need',"//div[@class='detail-content']//div[@class='text']/text()")
        itemloader.add_xpath('company_info',"//div[contains(@class,'company-info')]/div/text()")
        itemloader.add_xpath('addr',"//div[contains(@class,'location-address')]/text()")
        job_item = itemloader.load_item()
        yield job_item
```


#### models/mondo_db.py

```python
from mongoengine import connect
from mongoengine import Document, StringField, IntField, FloatField, ListField, EmbeddedDocument, EmbeddedDocumentField

connect('scrapy_test')    # 简写 连接数据库

class BossJob(Document):
    title = StringField(max_length=255,required=True)
    desc = StringField(max_length=255,required=True)
    salary = StringField(max_length=100,required=True)
    job_need = StringField(max_length=500,required=True)
    company_info = StringField(max_length=255,required=True)
    addr = StringField(max_length=100,required=True)

    meta = {
        'collection':'bossjob'  # 指定数据表
    }

    def __str__(self):
        return self.title
```




#### items.py

```python

import re
import scrapy
from scrapy.loader import ItemLoader
from scrapy.loader.processors import TakeFirst,MapCompose,Join


# ======== 定义预处理 去空格  =======

def remove_space(value):
    value = re.sub(r'\s+','',value)
    return value


class BossItemLoader(ItemLoader):
    default_output_processor = TakeFirst()  # 默认out_put_processor 取出第一个


class BossjobItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    title = scrapy.Field(input_processor=MapCompose(remove_space))    # input指定预处理函数
    desc = scrapy.Field(input_processor=MapCompose(remove_space),output_processor=Join()) 
    salary = scrapy.Field(input_processor=MapCompose(remove_space))
    job_need = scrapy.Field(input_processor=MapCompose(remove_space),output_processor=Join())
    company_info = scrapy.Field(input_processor=MapCompose(remove_space))
    addr = scrapy.Field(input_processor=MapCompose(remove_space))

```

#### pipelines.py

```python

from .models.mongo_db import BossJob

class BossjobPipeline(object):

    def process_item(self, item, spider):
        boosjob_obj = BossJob(
            title = item['title'],
            desc = item['desc'],
            salary = item['salary'],
            job_need=item['job_need'],
            company_info=item['company_info'],
            addr=item['addr'],
        )
        boosjob_obj.save()   # 保存数据到数据库
        return item
```

#### settings.py

当然还有settings中的配置

```python

ITEM_PIPELINES = {
   'bossjob.pipelines.BossjobPipeline': 300,
}

DEFAULT_REQUEST_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'en',
    'user-agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.140 Safari/537.36',
    'referer': 'https://www.zhipin.com/'
}

DOWNLOAD_DELAY = 3

ROBOTSTXT_OBEY = False

```

#### 补充一点 scrapy的入口文件  enterpoint.py

```python
import os,sys

from scrapy.cmdline import execute

sys.path.append(os.path.dirname(os.path.abspath(__file__)))

execute(['scrapy','crawl','job'])
# execute(['scrapy','crawl','job'，'nolog']) 没有日志打印
```


