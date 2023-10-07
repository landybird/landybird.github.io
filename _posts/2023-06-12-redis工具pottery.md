---
title: redis工具pottery
description: pottery是一个redis工具包，提供了redis的dict、set、list、deque、counter等数据结构的封装，使用起来更加方便。
categories:
- redis
tags:
- redis
---

<br>





### 使用方式

获取 `redis client`

```python
from redis import Redis 
redis = Redis.from_url("redis://10.0.0.207:6379/0")
```

#### Dict `RedisDict`

RedisDict 是一个 Redis 支持的容器，对应Python 的 dict 

```python
from pottery import RedisDict
tel = RedisDict(redis=redis, key='tel')
tel['jack'] = 4098
tel['sape'] = 4139

# tel = RedisDict({'jack': 4098, 'sape': 4139}, redis=redis, key='tel')


>>> from pottery import RedisDict
>>> tel = RedisDict({'jack': 4098, 'sape': 4139}, redis=redis, key='tel')
>>> tel['guido'] = 4127
>>> tel
RedisDict{'jack': 4098, 'sape': 4139, 'guido': 4127}
>>> tel['jack']
4098
>>> del tel['sape']
>>> tel['irv'] = 4127
>>> tel
RedisDict{'jack': 4098, 'guido': 4127, 'irv': 4127}
>>> list(tel)
['jack', 'guido', 'irv']
>>> sorted(tel)
['guido', 'irv', 'jack']
>>> 'guido' in tel
True
>>> 'jack' not in tel
False
>>>
```

`RedisDict` 两个关键字参数:
    
    第一个是 Redis 客户端

    第二个是字典的 Redis 键名称 key

    除此之外，可以像使用任何其他 Python dict 一样使用 RedisDict 方式。


注意： 键和值必须是 JSON 可序列化的




    
####  Sets `RedisSet`

RedisSet 是一个 Redis 支持的容器，与 Python 的 set

```python
>>> from pottery import RedisSet
>>> basket = RedisSet({'apple', 'orange', 'apple', 'pear', 'orange', 'banana'}, redis=redis, key='basket')
>>> sorted(basket)
['apple', 'banana', 'orange', 'pear']
>>> 'orange' in basket
True
>>> 'crabgrass' in basket
False

>>> a = RedisSet('abracadabra', redis=redis, key='magic')
>>> b = set('alacazam')
>>> sorted(a)
['a', 'b', 'c', 'd', 'r']
>>> sorted(a - b)
['b', 'd', 'r']
>>> sorted(a | b)
['a', 'b', 'c', 'd', 'l', 'm', 'r', 'z']
>>> sorted(a & b)
['a', 'c']
>>> sorted(a ^ b)
['b', 'd', 'l', 'm', 'r', 'z']
>>>
```

`.contains_many()`多个元素进行更有效包含关系判断

```python
print(tuple(basket.contains_many('apple', 'orange', "peach")))
# (True, True, False)

```

#### List `RedisList`


```python
>>> from pottery import RedisList
>>> squares = RedisList([1, 4, 9, 16, 25], redis=redis, key='squares')
>>> squares
RedisList[1, 4, 9, 16, 25]
>>> squares[0]
1
>>> squares[-1]
25
>>> squares[-3:]
[9, 16, 25]
>>> squares[:]
[1, 4, 9, 16, 25]
>>> squares + [36, 49, 64, 81, 100]
RedisList[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
>>>
```

注意： 

    Python list 使用数组实现。Redis 使用双向链表实现

        在 RedisList 的头部或尾部插入元素是快速的，O（1）
        但是，按索引访问 RedisList 元素很慢，O（n）
    
    推荐使用  RedisDeque



#### Counters `RedisCounter` -- `hash`

```python
from pottery import RedisCounter
c = RedisCounter('abracadabra', redis=redis, key='my-counter')
c.most_common(3)
# [('a', 5), ('b', 2), ('r', 2)]
c.clear()

c = RedisCounter(redis=redis, key='my-counter', a=4, b=2, c=0, d=-2)
>>> sorted(c.elements())
# ['a', 'a', 'a', 'a', 'b', 'b']

```


#### Deques `RedisDeque` -- `list`

```python   
>>> from pottery import RedisDeque
>>> d = RedisDeque('ghi', redis=redis, key='letters')
>>> for elem in d:
...     print(elem.upper())
G
H
I

>>> d.append('j')
>>> d.appendleft('f')
>>> d
RedisDeque(['f', 'g', 'h', 'i', 'j'])

>>> d.pop()
'j'
>>> d.popleft()
'f'
>>> list(d)
['g', 'h', 'i']
>>> d[0]
'g'
>>> d[-1]
'i'

>>> list(reversed(d))
['i', 'h', 'g']
>>> 'h' in d
True
>>> d.extend('jkl')
>>> d
RedisDeque(['g', 'h', 'i', 'j', 'k', 'l'])
>>> d.rotate(1)
>>> d
RedisDeque(['l', 'g', 'h', 'i', 'j', 'k'])
>>> d.rotate(-1)
>>> d
RedisDeque(['g', 'h', 'i', 'j', 'k', 'l'])

>>> RedisDeque(reversed(d), redis=redis)
RedisDeque(['l', 'k', 'j', 'i', 'h', 'g'])
>>> d.clear()

>>> d.extendleft('abc')
>>> d
RedisDeque(['c', 'b', 'a'])
>>>
```


#### Queues `RedisSimpleQueue` -- `list`

`RedisSimpleQueue` 是一个 Redis 支持的多生产者、多消费者 FIFO 队列，与 Python 的 `queue.SimpleQueue` 类似

    
    In general

        use a Python queue.Queue if you’re using it in one or more threads
    
        use multiprocessing.Queue if you’re using it between processes

        use RedisSimpleQueue if you’re sharing it across machines or if 
        you need for your queue to persist across application crashes or restarts.

```python
>>> from pottery import RedisSimpleQueue
>>> cars = RedisSimpleQueue(redis=redis, key='cars')
>>> cars.empty()
True
>>> cars.qsize()
0
>>> cars.put('Jeep')
>>> cars.put('Honda')
>>> cars.put('Audi')
>>> cars.empty()
False
>>> cars.qsize()
3
>>> cars.get()
'Jeep'
>>> cars.get()
'Honda'
>>> cars.get()
'Audi'
>>> cars.empty()
True
>>> cars.qsize()
0
>>>
``` 

#### Redlock 

`Redlock` 是一种安全可靠的锁，用于协调对跨线程、进程甚至机器共享的资源的访问，而不会出现单点故障。

[基本原理和算法描述](https://redis.io/topics/distlock/)

```python
>>> from pottery import Redlock
>>> printer_lock = Redlock(key='printer', masters={redis}, auto_release_time=.2)


>>> if printer_lock.acquire():
...     # Critical section - print stuff here.
...     print('printer_lock is locked')
...     printer_lock.release()
printer_lock is locked
>>> bool(printer_lock.locked())
False
>>> with printer_lock:
...     # Critical section - print stuff here.
...     print('printer_lock is locked')
printer_lock is locked
>>> bool(printer_lock.locked())
False
>>>


>>> printer_lock_1.acquire()
True
>>> printer_lock_2.acquire(timeout=printer_lock_1.auto_release_time / 2)  # Waits 100 milliseconds.
False
>>> import contextlib
>>> from pottery import ReleaseUnlockedLock
>>> with contextlib.suppress(ReleaseUnlockedLock):
...     printer_lock_1.release()


```

在生产中，应该有 5 个 Redis master


    
    Keyword arguments:
        key -- 标识资源的字符串
        masters -- the Redis clients used to achieve quorum for this
            Redlock's state
        raise_on_redis_errors -- whether to raise the QuorumIsImplssible
            exception when too many Redis masters throw errors
        auto_release_time -- the timeout in seconds by which to
            automatically release this Redlock, unless it's already been
            released
        num_extensions -- the number of times that this Redlock's lease can
            be extended
        context_manager_blocking -- when using this Redlock as a context
            manager, whether to block when acquiring
        context_manager_timeout -- if context_manager_blocking, how long to
            wait when acquiring before giving up and raising the
            QuorumNotAchieved exception


<br>

----

- `synchronize()`

`synchronize()` 是一个装饰器，使用`Redlock` 一次只允许一个线程执行一个函数

```python
>>> from pottery import synchronize
>>> @synchronize(key='synchronized-func', masters={redis}, auto_release_time=1.5, blocking=True, timeout=-1)
... def func():
...   # Only one thread can execute this function at a time.
...   return True
...
>>> func()
True
```


#### AIORedlock  `asyncio.Lock`

```python

>>> import asyncio
>>> from redis.asyncio import Redis as AIORedis
>>> from pottery import AIORedlock
>>> async def main():
...     aioredis = AIORedis.from_url('redis://localhost:6379/1')
...     shower = AIORedlock(key='shower', masters={aioredis})
...     if await shower.acquire():
...         # Critical section - no other coroutine can enter while we hold the lock.
...         print(f"shower is {'occupied' if await shower.locked() else 'available'}")
...         await shower.release()
...     print(f"shower is {'occupied' if await shower.locked() else 'available'}")
...
>>> asyncio.run(main(), debug=True)
shower is occupied
shower is available



>>> asyncio.set_event_loop(asyncio.new_event_loop())
>>> async def main():
...     aioredis = AIORedis.from_url('redis://localhost:6379/1')
...     shower = AIORedlock(key='shower', masters={aioredis})
...     async with shower:
...         # Critical section - no other coroutine can enter while we hold the lock.
...         print(f"shower is {'occupied' if await shower.locked() else 'available'}")
...     print(f"shower is {'occupied' if await shower.locked() else 'available'}")
...
>>> asyncio.run(main(), debug=True)
shower is occupied
shower is available

```


#### NextID [Rationale and algorithm description.](http://antirez.com/news/102)

NextID 安全可靠地跨线程、进程甚至机器生成不断增加的 ID，而不会发生单点故障。

```python
>>> from pottery import NextID
>>> tweet_ids = NextID(key='tweet-ids', masters={redis})

>>> next(tweet_ids)
1
>>> next(tweet_ids)
2
>>> next(tweet_ids)

```

两个注意事项:

    如果许多客户端同时生成 ID，则 ID 序列中可能存在“漏洞”（例如：1、2、6、10、11、21 等
    
    此算法可扩展到每秒约 5，000 个 ID（使用 5 个 Redis 主节点）。如果您需要比这更快的 ID，那么您可能需要考虑其他技术。



#### redis_cache() 


```python 
@redis_cache(redis=redis, key="expensive_function", timeout=10)
def expensive_function(n):
    time.sleep(n)
    return n


# sleep 5
print(expensive_function(5))

# 直接返回
print(expensive_function(5))

print(expensive_function.cache_info())
```



#### Bloom filters 🌸

布隆过滤器是一种强大的数据结构, 用于解决 `我以前见过这个元素吗` 

    布隆过滤器是概率性的 

    但是他们永远不会产生假阴性（所以每次他们报告你以前没有见过某个特定的元素时，你一定从未见过它）
        如果计算结果为 False ，则 bf 一定没有插入它


```python

redis = Redis.from_url("redis://localhost:6379/10")

dilberts = BloomFilter(num_elements=100,  false_positives=0.01, redis=redis, key="dilberts")
dilberts.add('rajiv')

dilberts.update({'raj', 'dan'})

print(tuple(dilberts.contains_many('rajiv', 'raj', 'dan', 'luis')))
```


#### HyperLogLogs 🪵

`HyperLogLogs`是一种有趣的数据结构，旨在回答`我见过多少个不同的元素`这个问题。

```python

from pottery import HyperLogLog
google_searches = HyperLogLog(redis=redis, key='google-searches')

google_searches.update({
     'google in 1998',
     'google in 1998',
     'google in 1998',
     'minesweeper',
     'joey tribbiani',
     'wizard of oz',
     'rgb to hex',
     'pac-man',
     'breathing exercise',
     'do a barrel roll',
     'snake',
 })

print(len(google_searches)) 
# 9 (not 11)

```


HyperLogLog 主要适用于元素数量非常大的情况，如果只是几个元素，直接用 Python 的 set 会更简单高效



