---
title: asyncio.gather vs asyncio.wait
description: python 中 asyncio gather 和 wait的比较 
categories:
- python
tags:
- python基础
---

<br>

# asyncio.gather vs asyncio.wait

<br>
两个方法都是运行并且获取tasks的结果， 但是有自己的独特使用方式:

> asyncio.gather() `waits on a bunch of futures and return their results in a given order.`

Returns a Future instance, allowing high level grouping of tasks:

```python

import asyncio
from pprint import pprint

import random


async def coro(tag):
    print(">", tag)
    await asyncio.sleep(random.uniform(1, 3))
    print("<", tag)
    return tag


loop = asyncio.get_event_loop()

group1 = asyncio.gather(*[coro("group 1.{}".format(i)) for i in range(1, 6)])
group2 = asyncio.gather(*[coro("group 2.{}".format(i)) for i in range(1, 4)])
group3 = asyncio.gather(*[coro("group 3.{}".format(i)) for i in range(1, 10)])

all_groups = asyncio.gather(group1, group2, group3)

results = loop.run_until_complete(all_groups)

loop.close()

pprint(results)

```

All tasks in a group can be cancelled by calling group2.cancel() or even all_groups.cancel(). See also .gather(..., return_exceptions=True),


>  asyncio.wait() `gives done and pending tasks, have to mannually collect the values.`

Supports waiting to be stopped after the first task is done, or after a specified timeout, allowing lower level precision of operations:

```python

import asyncio
import random


async def coro(tag):
    print(">", tag)
    await asyncio.sleep(random.uniform(0.5, 5))
    print("<", tag)
    return tag


loop = asyncio.get_event_loop()

tasks = [coro(i) for i in range(1, 11)]

print("Get first result:")
finished, unfinished = loop.run_until_complete(
    asyncio.wait(tasks, return_when=asyncio.FIRST_COMPLETED))

for task in finished:
    print(task.result())
print("unfinished:", len(unfinished))

print("Get more results in 2 seconds:")
finished2, unfinished2 = loop.run_until_complete(
    asyncio.wait(unfinished, timeout=2))

for task in finished2:
    print(task.result())
print("unfinished2:", len(unfinished2))

print("Get all other results:")
finished3, unfinished3 = loop.run_until_complete(asyncio.wait(unfinished2))

for task in finished3:
    print(task.result())

loop.close()

```

get_event_loop will try to access any available event loop, if there is not, it will call new_event_loop with set_event_loop

Simply use asyncio.gather when you won't do any further actions individual task. Use gather provides you a way to stop whole task group as well, see this SO answer.

You may notice in the following, () generator is used instead of [] list comprehension, as you don't need the list actually be saved in memory, and you don't need it until iteration. Change back to [] bracket if it is not the case.

```
Whole working example will be:

# len(ad_accounts) = 1000 for example

chunk_size = 100
batched_tasks = (ad_accounts[i:i + chunk_size] for i in range(0, len(ad_accounts), chunk_size))
_loop = asyncio.get_event_loop()

for task_group in batched_tasks:
    task_list = [
            asyncio.ensure_future(_handle_account(account)) for ad_account in task_group
	        ]
    #just submit 100 tasks here once
    _loop.run_until_complete(asyncio.gather(*task_list))
    # Or _loop.run_until_complete(asyncio.wait(task_list))

```
