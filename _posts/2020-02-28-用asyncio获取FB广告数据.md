---
title: 用asyncio获取FB广告数据                                             
description: 信号量 + queue 并发请求 
categories:
- python
tags:
- python   
---


```python
# coding: utf-8

import asyncio
import aiohttp
import json
import xlrd
import aiomysql
import os

BASE_URL = "https://graph.facebook.com"
VERSION = "v4.0"
BM_ID = "BM_ID"
TOKEN = "TOKEN"

USE_PROXY = True

proxy = 'proxy'
os.environ["https_proxy"] = proxy

time_range = json.dumps(
    {"since": "2018-01-01", "until": "2018-12-31"}
)

owned_ad_accounts_url = f"{BASE_URL}/{VERSION}/{BM_ID}/owned_ad_accounts?limit=500&access_token={TOKEN}"
insights_api_account_url = 'act_{account_id}/insights?fields=account_id,spend,clicks,impressions,cpc,cpm,cpp,ctr,actions,action_values&time_range={time_range}&breakdowns=country&limit=500'

pattern_insights_url = '{ad_id}?fields=account_id, adset{{id, bid_strategy, optimization_goal, targeting}},campaign{{objective, bid_strategy}},adcreatives{{video_id, image_hash}}, insights.fields(spend,clicks,impressions,actions,action_values).breakdowns(country).limit(400).time_range({time_range})&limit=5'
final_insights_url = f"{BASE_URL}/{VERSION}/{pattern_insights_url}&access_token={TOKEN}"

pattern_checkout_insights_url = 'act_{account_id}?fields=insights.fields(spend,clicks,impressions,actions,action_values).time_range({time_range})'
checkout_insights_url = f"{BASE_URL}/{VERSION}/{pattern_checkout_insights_url}&access_token={TOKEN}"

pattern_checkout_ad_insights_url = '{ad_id}?fields=insights.fields(spend,clicks,impressions,actions,action_values).time_range({time_range})'
checkout_ad_insights_url = f"{BASE_URL}/{VERSION}/{pattern_checkout_ad_insights_url}&access_token={TOKEN}"


class UrlEmptyError(Exception):
    pass


class AioMysql:

    def __init__(self):
        ...

    async def register(self):
        '''
        :return:
        '''
        try:
            pool = await aiomysql.create_pool(host='host', port=3306,
                                              user='user', password='pwd',
                                              db='db', charset='utf8')
            return pool

        except asyncio.CancelledError:
            raise asyncio.CancelledError

        except Exception as ex:
            print(f"mysql连接异常: {ex.args[0]}")
            return False

    async def getCurosr(self, pool):
        '''
        :return:
        '''
        conn = await pool.acquire()
        cur = await conn.cursor()
        return conn, cur

    async def batchInsert(self, conn, cur, values):

        sql = """
          insert ignore into db.table(
                    field1,
                    field2,
                    field3,
                    ...
                  ) values(%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s,%s)
          """

        await cur.executemany(sql, values)
        await conn.commit()
        print(f"saved: {cur.rowcount}")
        return cur.rowcount

    async def query(self, cur, sql):
        await cur.execute(sql)
        ret = await cur.fetchall()
        return set(
            [
                item[0] for item in ret
            ]
        )

    async def query_field(self, cur, account_id, field):
        sql = f"select sum({field})  from table where account_id = %s"
        await cur.execute(sql, f"{account_id}")
        ret = (await cur.fetchone())[0]
        return ret


def data_ad_parse(data):
    final_ad_info_list = []
    unit_info = data
    campaign_info = unit_info.get("campaign", {})
    adset_info = unit_info.get("adset", {})
    ad_id = unit_info.get("id", '')
    account_id = unit_info.get("account_id", '')
    adcreatives_info = unit_info.get("adcreatives", {})
    insights_info = unit_info.get("insights", {})

    # campaign_id
    campaign_id = campaign_info.get("id", '')
    objective = campaign_info.get("objective", '')
    campaign_bid_strategy = campaign_info.get("bid_strategy", '')

    # adset_info
    adset_id = adset_info.get("id", '')
    optimization_goal = adset_info.get("optimization_goal", '')
    targeting = adset_info.get("targeting", {})
    adset_bid_strategy = adset_info.get("bid_strategy", '')

    flexible_spec = json.dumps(targeting.get("flexible_spec", []))
    publisher_platforms = json.dumps(targeting.get("publisher_platforms", []))
    device_platforms = json.dumps(targeting.get("device_platforms", []))
    custom_audiences = json.dumps(targeting.get("custom_audiences", []))

    # geo_locations = targeting.get("geo_locations", {})
    # countries = json.dumps(geo_locations.get("countries", []))

    # adcreative_info
    image_hash = video_id = ''
    adcreatives_info_data = adcreatives_info.get("data", [])
    if adcreatives_info_data:
        image_hash = adcreatives_info_data[0].get("image_hash", '')
        video_id = adcreatives_info_data[0].get("video_id", '')

    insights_data = insights_info.get('data', [])
    if insights_data:
        for insight_ in insights_data:
            # insights_info
            spend = clicks \
                = impressions \
                = installs \
                = mobile_app_purchases \
                = website_purchases \
                = website_purchases_conversion_value \
                = mobile_app_purchases_conversion_value \
                = ''

            insights_data = insight_
            spend = insights_data.get("spend", '')
            country = insights_data.get("country", '')
            clicks = insights_data.get("clicks", '')
            impressions = insights_data.get("impressions", '')
            actions = insights_data.get("actions", [])
            action_values = insights_data.get("action_values", [])

            if actions:
                for action in actions:
                    if action.get("action_type") == 'mobile_app_install':
                        installs = action.get("value", '')

                    if action.get("action_type") == 'app_custom_event.fb_mobile_purchase':
                        mobile_app_purchases = action.get("value", '')

                    if action.get("action_type") == 'onsite_web_purchase':
                        website_purchases = action.get("value", '')

            if action_values:
                for action_type in action_values:

                    if action_type.get("action_type") == 'app_custom_event.fb_mobile_purchase':
                        mobile_app_purchases_conversion_value = action_type.get("value", '')

                    if action_type.get("action_type") == 'onsite_web_purchase':
                        website_purchases_conversion_value = action_type.get("value", '')

            final_ad_info_list.append(
                {
                    "account_id": account_id,
                    "campaign_id": campaign_id,
                    "adset_id": adset_id,
                    "id": ad_id,
                    "impressions": impressions,
                    "clicks": clicks,
                    "spend": spend,
                    "installs": installs,
                    "mobile_app_purchases": mobile_app_purchases,
                    "website_purchases": website_purchases,
                    "website_purchases_conversion_value": website_purchases_conversion_value,
                    "mobile_app_purchases_conversion_value": mobile_app_purchases_conversion_value,
                    "country": country,
                    "custom_audiences": custom_audiences,
                    "flexible_spec": flexible_spec,
                    "publisher_platforms": publisher_platforms,
                    "device_platforms": device_platforms,
                    "adset_bid_strategy": adset_bid_strategy,
                    "optimization_goal": optimization_goal,
                    "objective": objective,
                    "campaign_bid_strategy": campaign_bid_strategy,
                    "video_id": video_id,
                    "image_hash": image_hash,

                }
            )

    url = data['paging']['next'] if 'paging' in data and 'next' in data['paging'] else ''
    return url, final_ad_info_list


def gen_list(account_ids, limit=5000):
    for i in range(0, len(account_ids), limit):
        yield account_ids[i: i + limit]


def get_all_accounts(file_name='excel.xlsx'):
    wb = xlrd.open_workbook(file_name)
    table = wb.sheets()[0]
    return table.col_values(0)[1:]


async def get_all_accounts(queue):
    # todo read excel

    wb = xlrd.open_workbook("excel1.xlsx")
    table = wb.sheets()[0]
    account_ids = table.col_values(0)[1:]
    for id in account_ids:
        await queue.put(id)


async def async_get_proxy(session, sem, url):
    global proxy
    async with sem:
        async with session.get(url, proxy=proxy) as resp:
            ret = await resp.json(content_type=None)
            # @todo Attempt to decode JSON with unexpected mimetype:
            return ret, url


async def async_get(session, sem, url):
    async with sem:
        async with session.get(url) as resp:
            ret = await resp.json(content_type=None)
            return ret, url


async def as_batch_post_proxy(session, url, sem):
    global proxy
    async with sem:
        async with session.post(url, proxy=proxy) as resp:
            response_ret = await resp.json()
            return response_ret


async def get_and_save_ad_info(ad_id, session, sem, mysql_client, mysql_pool):
    global final_insights_url
    global checkout_ad_insights_url
    url = final_insights_url.format(ad_id=ad_id, time_range=time_range)
    check_url = checkout_ad_insights_url.format(ad_id=ad_id, time_range=time_range)
    check_ret, _ = await async_get_proxy(session, sem, check_url)
    total_count = 0
    if check_ret.get("insights"):
        while 1:
            try:
                if not url:
                    raise UrlEmptyError()
                if USE_PROXY:
                    pre_ret, ori_url = await async_get_proxy(session, sem, url)
                else:
                    pre_ret, ori_url = await async_get(session, sem, url)

                url, ad_info_list = data_ad_parse(pre_ret)
                total_count += len(ad_info_list)

                if ad_info_list:
                    try:
                        insert_list = []
                        for item in ad_info_list:
                            insert_list.append(
                                (
                                    item["id"],
                                    item["account_id"],
                                    item["campaign_id"],
                                    item["adset_id"],
                                    item["impressions"],
                                    item["spend"],
                                    item["clicks"],
                                    item["installs"],
                                    item["mobile_app_purchases"],
                                    item["website_purchases"],
                                    item["website_purchases_conversion_value"],
                                    item["mobile_app_purchases_conversion_value"],
                                    item["country"],
                                    item["custom_audiences"],
                                    item["flexible_spec"],
                                    item["publisher_platforms"],
                                    item["device_platforms"],
                                    item["adset_bid_strategy"],
                                    item["optimization_goal"],
                                    item["objective"],
                                    item["campaign_bid_strategy"],
                                    item["video_id"],
                                    item["image_hash"]
                                )
                            )

                        async with mysql_pool.acquire() as conn:
                            async with conn.cursor() as cur:
                                await mysql_client.batchInsert(conn, cur, insert_list)
                    except:
                        import traceback as tb
                        tb.print_exc()


            except UrlEmptyError:
                # print('url is null get account done{}'.format(e))
                break
            except Exception:
                import traceback
                traceback.print_exc()

    return total_count


async def consume_account_queue(queue, session, sem, mysql_client, mysql_pool):
    ret = []
    while 1:
        ad_id = await queue.get()
        if not ad_id:
            print("empty")
            queue.task_done()
            return ret
        else:
            total_count = await get_and_save_ad_info(ad_id, session, sem, mysql_client, mysql_pool)
            print(f"{ad_id} end with {total_count}")
            queue.task_done()


async def get_accounts_with_data(account_id, session, sem):
    check_url = checkout_insights_url.format(account_id=account_id, time_range=time_range)
    check_ret, _ = await async_get_proxy(session, sem, check_url)
    if check_ret.get("insights"):
        return account_id


async def root():
    try:
        queue = asyncio.Queue()

        mysql_client = AioMysql()
        mysql_pool = await mysql_client.register()

        sem = asyncio.Semaphore(300)
        conn1 = aiohttp.TCPConnector()
        conn2 = aiohttp.TCPConnector()
        timeout = aiohttp.ClientTimeout(total=None)


        query_all_account = "select distinct account_id from table1"

        async with mysql_pool.acquire() as mysql_conn:
            async with mysql_conn.cursor() as cur:
                undealed_accounts = await mysql_client.query(cur, query_all_account)

        async with aiohttp.ClientSession(connector=conn1, timeout=timeout) as session:
            undealed_accounts_with_data_tasks = [get_accounts_with_data(account_id, session, sem) for account_id in
                                                 undealed_accounts]
            undealed_accounts_results = await asyncio.gather(*undealed_accounts_with_data_tasks)

        undealed_accounts_with_data_list = list(filter(lambda item: item, undealed_accounts_results))
        undealed_accounts_with_data_list_str = ",".join(undealed_accounts_with_data_list)

        query_ad_with_data_sql = "select ad_id from table1 where account_id in (%s)" % (
            undealed_accounts_with_data_list_str)
        async with mysql_pool.acquire() as mysql_conn:
            async with mysql_conn.cursor() as cur:
                undealed_ads = await mysql_client.query(cur, query_ad_with_data_sql)

        query_dealed_ads_with_data_sql = 'select id from table2'
        async with mysql_pool.acquire() as mysql_conn:
            async with mysql_conn.cursor() as cur:
                dealed_ads = await mysql_client.query(cur, query_dealed_ads_with_data_sql)

        undealed_ads = set(undealed_ads) - set(dealed_ads)

        #
        print(f"undealed_accounts: {len(undealed_ads)}")

        for id in undealed_ads:
            queue.put_nowait(id)

        coro_num = 200
        tasks = []
        async with aiohttp.ClientSession(connector=conn2, timeout=timeout) as session:
            for i in range(coro_num):
                task = asyncio.ensure_future(consume_account_queue(queue, session, sem, mysql_client, mysql_pool))
                tasks.append(task)

            await queue.join()
            print("queue processed")

            for task in tasks:
                task.cancel()

        mysql_pool.close()
        print("pool closed")
        await mysql_pool.wait_closed()
        print("pool waited")

    except:
        import traceback as tb
        tb.print_exc()


def main():
    loop = asyncio.get_event_loop()
    loop.run_until_complete(root())
    loop.close()


if __name__ == '__main__':
    main()
```
