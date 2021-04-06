---
title: python调用翻译api   
description: 利用baidu, youdao, google翻译
categories:
- python
tags:
- python   
---


#### 自动对文本进行翻译

[>> 文档youdao](https://ai.youdao.com/console/#/service-singleton/text-translation)

[>> 文档baidu](https://api.fanyi.baidu.com/api/trans/product/desktop?req=detail)

[>> 文档google](https://cloud.google.com/translate/docs/basic/quickstart?hl=zh-CN)

```python
# -*- coding: utf-8 -*-
import sys
import uuid
import requests
import hashlib
import time
import os
import re
import logging
import random
import string
import json
import datetime
from hashlib import md5
from enum import IntEnum
from importlib import reload
from concurrent.futures import ThreadPoolExecutor

from requests import Session
from django.conf import settings
from django.core.management.base import BaseCommand
from googletrans import Translator
from fake_useragent import UserAgent

from .models import BaseModel
from .enums import OEReviewStatusEnum

reload(sys)

if settings.DEBUG:
    os.environ['https_proxy'] = 'http://...'

logger = logging.getLogger('aor')
logger.setLevel("INFO")

ZH_COMPILE = re.compile(r"[\u4e00-\u9fa5]")
UA = UserAgent().msie

Date = datetime.datetime.now()


class TranslatorEnum(IntEnum):
    GOOGLE_TRANS = 1
    GOOGLE = 2
    BAIDU = 3
    YOUDAO = 4



def encrypt(signStr):
    hash_algorithm = hashlib.sha256()
    hash_algorithm.update(signStr.encode('utf-8'))
    return hash_algorithm.hexdigest()


def truncate(q):
    if q is None:
        return None
    size = len(q)
    return q if size <= 20 else q[0:10] + str(size) + q[size - 10:size]


def do_request(data):
    headers = {'Content-Type': 'application/x-www-form-urlencoded'}
    return requests.post(settings.YOUDAO_TRANS_URL, data=data, headers=headers)


def _youdao_translate(origin_entity):
    origin_entity_name = origin_entity.entity_name_zh
    q = origin_entity_name
    data = {}
    data['from'] = 'auto'
    data['to'] = 'en'
    data['signType'] = 'v3'
    curtime = str(int(time.time()))
    data['curtime'] = curtime
    salt = str(uuid.uuid1())
    signStr = settings.YOUDAO_TRANS_APP_KEY + truncate(q) + salt + curtime + settings.YOUDAO_TRANS_SECRET_KEY
    sign = encrypt(signStr)
    data['appKey'] = settings.YOUDAO_TRANS_APP_KEY
    data['q'] = q
    data['salt'] = salt
    data['sign'] = sign
    id = origin_entity.id

    # data['vocabId'] = "用户指定的词典 out_id，目前支持英译中"
    try:
        response = do_request(data)
        r = json.loads(response.text)
        entity_name_en = r["translation"][0]
        logger.info(
            "【Youdao_Translate】translate string: {}, size: {}.".format(origin_entity_name, len(origin_entity_name)))
        BaseModel.objects.filter(id=id).update(entity_name_en=entity_name_en, translate_time=Date)

    except Exception:
        import traceback as tb; tb.print_exc()
        ...

def _googletrans_translate(origin_entity):
    origin_entity_name = origin_entity.entity_name_zh
    src = 'en'
    if ZH_COMPILE.search(origin_entity_name):
        src = 'zh-CN'
    id = origin_entity.id
    if origin_entity_name:
        try:
            translate = Translator(user_agent=UA)
            print(origin_entity_name)
            result = translate.translate(origin_entity_name, src=src, dest='en')
            BaseModel.objects.filter(id=id).update(entity_name_en=result.text, translate_time=Date)

        except Exception:
            import traceback as tb; tb.print_exc()
            ...

def _baidu_translate(origin_entity):
    origin_entity_name = origin_entity.entity_name_zh
    base_url = settings.BAIDU_TRANS_URI
    ran_str = ''.join(random.sample(string.ascii_letters + string.digits, 8))
    q = origin_entity_name
    from_ = "auto"
    to_ = "en"
    app_id = settings.BAIDU_TRANS_APP_ID
    secret_key = settings.BAIDU_TRANS_SECRET_KEY

    str_1 = app_id + q + ran_str + secret_key
    h1 = md5()
    h1.update(str_1.encode("utf-8"))
    param = {
        "q": q,
        "from": from_,
        "to": to_,
        "appid": app_id,
        "sign": h1.hexdigest(),
        "salt": ran_str,
    }

    id = origin_entity.id
    if origin_entity_name:
        try:
            r = Session().get(base_url, params=param)
            entity_name_en = json.loads(r.text)["trans_result"][0]["dst"]
            logger.info("【Baidu_Translate】translate string: {}, size: {}.".format(origin_entity_name, len(origin_entity_name)))
            BaseModel.objects.filter(id=id).update(entity_name_en=entity_name_en, translate_time=Date)

        except Exception:
            import traceback as tb; tb.print_exc()
            ...






class BaseTranslator(object):
    def __init__(self, translator, batch_size=1):
        self.translator = translator
        self.batch_size = batch_size

    def translate(self, queryset):
        if self.translator == TranslatorEnum.BAIDU:
            translate = _baidu_translate
        elif self.translator == TranslatorEnum.GOOGLE_TRANS:
            translate = _googletrans_translate
        elif self.translator == TranslatorEnum.YOUDAO:
            translate = _youdao_translate
        else:
            translate = _baidu_translate
        origin_entity_list = [queryset[i: i + self.batch_size] for i in
                              range(0, len(queryset), self.batch_size)]
        for batch_entity in origin_entity_list:
            with ThreadPoolExecutor(max_workers=10) as executor:
                executor.map(translate, batch_entity)

            time.sleep(1)




class Command(BaseCommand):
    help = '自动更新entity英文名称'

    def add_arguments(self, parser):
        ...
        # 指定settings   python manage.py schedule_task --pythonpath path/to/settings --settings settings

    def handle(self, *args, **options):
        time_start = time.perf_counter()
        try:
            pending_entity_queryset = BaseModel.objects.\
                                    filter(entity_name_en='')[:10000]
            # BaseTranslator(TranslatorEnum.BAIDU, batch_size=10).translate(pending_entity_queryset)
            BaseTranslator(TranslatorEnum.YOUDAO, batch_size=10000).translate(pending_entity_queryset)
            # BaseTranslator(TranslatorEnum.GOOGLE_TRANS, batch_size=10).translate(pending_entity_queryset)


        except Exception:
            import traceback as tb; tb.print_exc()
            logger.error(tb.format_exc())

        logger.info("aor finished cost: {time}".format(time = time.perf_counter() - time_start))




```
