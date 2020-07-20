---
title: redis stream数据结构(`redis5.0+`)                     
description: stream
categories:
- redis
tags:
- redis   
---



#### redis stream 的基本概念 

`Streams` are `append-only data-structures` that store a `unique identifier (typically a timestamp)` along with `arbitrary key/value data`

`stream`可以是一个看起来比`pubsub`更可靠的`消息队列`

    参考了kafka的 消费组模型


    

这里使用 [ walrus ](https://walrus.readthedocs.io/en/latest/) 来对redis进行操作

    pip install walrus
     
    stream 的两种操作

    
        standalone-mode: 
    
            streams act much like every other data-structure
    
        consumer-groups: 
            
            streams become stateful, with state such as “which messages were read?”, “who read what?”, etc are tracked within Redis
        

|操作|说明|
|---|---|
|`Stream.add()`|add a new item (XADD)|
|`Stream.range()`|Read a range of items (XRANGE)|
|`Stream.read()`|Read new messages, optionally blocking (XREAD)|
|`Stream.length()`|Delete one or more items (XDEL)|
|`Stream.range()`|Get the length of the stream (XLEN)|
|`Stream.trim()`|Trim the length to a given size (XTRIM)|
|`Stream.set_id()`|Set the maximum allowable ID (XSETID)|


```python
In [2]: from walrus import Database


# 初始化 stream 对象
In [3]: db = Database()
In [4]: stream = db.Stream("stream-a")
In [5]: msgid = stream.add({"message":"hello, stream"})

# In [6]: print(msgid)
# b'1595226396583-0'

# 增加消息
In [7]: msgid2 = stream.add({"message": "message1"})
In [8]: msgid3 = stream.add({"message": "message2"})



# 切片获取
In [11]: messages = stream[msgid2:]

#In [12]: messages
#Out[12]:
#[(b'1595226471740-0', {b'message': b'message1'}),
# (b'1595226478480-0', {b'message': b'message2'})]


# 指定步长
In [15]: messages = stream[msgid::2]

# In [16]: messages
# Out[16]:
#[(b'1595226396583-0', {b'message': b'hello, stream'}),
#(b'1595226471740-0', {b'message': b'message1'})]


# 删除消息
In [18]: stream.delete(msgid4)
# 或者
In [22]: del stream[msgid2]


# 关于trim的使用

# Add 1000 items to "stream-2".
stream2 = db.Stream('stream-2')
for i in range(1000):
    stream2.add({'data': 'message-%s' % i})

# Trim stream-2 to (approximately) 10 most-recent messages.
nremoved = stream2.trim(10)
print(nremoved)
# 909
print(len(stream2))
# 91

# To trim to an exact number, specify `approximate=False`:
stream2.trim(10, approximate=False)  # Returns 81.
print(len(stream2))
# 10



#  读取消息

(1) stream.read()

# Returns:
[(b'1539008903588-0', {b'message': b'hello, stream'}),
 (b'1539008914283-0', {b'message': b'message 2'}),
 (b'1539008918230-0', {b'message': b'message 3'})]
 
 
 
(2) stream.read(last_id='')
 
 

In [35]: stream.read()
Out[35]:
[(b'1595227107788-1', {b'data': b'message-990'}),
 (b'1595227107788-2', {b'data': b'message-991'}),
 (b'1595227107788-3', {b'data': b'message-992'}),
 (b'1595227107788-4', {b'data': b'message-993'}),
 (b'1595227107788-5', {b'data': b'message-994'}),
 (b'1595227107788-6', {b'data': b'message-995'}),
 (b'1595227107788-7', {b'data': b'message-996'}),
 (b'1595227107788-8', {b'data': b'message-997'}),
 (b'1595227107789-0', {b'data': b'message-998'}),
 (b'1595227107789-1', {b'data': b'message-999'})]

In [36]: stream.read(last_id='1595227107788-1')
Out[36]:
[(b'1595227107788-2', {b'data': b'message-991'}),
 (b'1595227107788-3', {b'data': b'message-992'}),
 (b'1595227107788-4', {b'data': b'message-993'}),
 (b'1595227107788-5', {b'data': b'message-994'}),
 (b'1595227107788-6', {b'data': b'message-995'}),
 (b'1595227107788-7', {b'data': b'message-996'}),
 (b'1595227107788-8', {b'data': b'message-997'}),
 (b'1595227107789-0', {b'data': b'message-998'}),
 (b'1595227107789-1', {b'data': b'message-999'})]

In [37]: stream.read(last_id='1595227107788-8')
Out[37]:
[(b'1595227107789-0', {b'data': b'message-998'}),
 (b'1595227107789-1', {b'data': b'message-999'})]

In [38]: stream.read(last_id='1595227107789-1')
Out[38]: []


# read block last_id = "$"
(3) 
# (provided no messages are added while waiting).
stream.read(block=2000, last_id='$')
# This will block for 2 seconds, after which an empty list is returned




# 消费者组

消费者组使实现强大的消息处理管道变得容易。 
消费者组允许应用程序从一个或多个流中读取，
同时跟踪读取了哪些消息，读取了哪些消息，
（确认）了这些消息。 可以检查和声明未确认的消息，从而简化“重试”逻辑




# Consumer groups require that a stream exist before the group can be
# created, so we have to add an empty message.
stream_keys = ['stream-a', 'stream-b', 'stream-c']
for stream in stream_keys:
    db.xadd(stream, {'data': ''})

# Create a consumer-group for streams a, b, and c. We will mark all
# messages as having been processed, so only messages added after the
# creation of the consumer-group will be read.
cg = db.consumer_group('cg-abc', stream_keys)
cg.create()  # Create the consumer group.
cg.set_id('$')



# Returns an empty list:
resp = cd.read() # 只读创建消费者组之后的消息
[]




# 消费者已经读到信息， 但是没有认知

# What messages have been read, but were not acknowledged, from stream c?
cg.stream_c.pending()

# Returns list of metadata, consisting of each pending message id, 
# consumer, message age, delivery count:
[{'message_id': b'1539023088126-0', 'consumer': b'cg-abc.c1',
  'time_since_delivered': 51329, 'times_delivered': 1}],
 {'message_id': b'1539023088126-1', 'consumer': b'cg-abc.c1',
  'time_since_delivered': 43772, 'times_delivered': 1},
 {'message_id': b'1539023088126-2', 'consumer': b'cg-abc.c2',
  'time_since_delivered': 5966, 'times_delivered': 1}]
  
  
#   主动认知 消息
cg2.stream_a.ack("1595234162885-0")



#   修改消息的归属
# claim pending messages, which transfers ownership of the message 

cg2.stream_a.claim("1595234089435-0")



# 查看信息 

db.Stream("stream-c").info()

#   删除

db.Stream("stream-c").delete("213213213-0")
db.delete("stream-c")


cg.destroy()






# 支持 time 时间序列

# Create a time-series consumer group named "demo-ts" for the
# streams s1 and s2.
ts = db.time_series('demo-ts', ['s1', 's2'])

# Add dummy data and create the consumer group.
db.xadd('s1', {'': ''}, id='0-1')
db.xadd('s2', {'': ''}, id='0-1')
ts.create()
ts.set_id('$')  # Do not read the dummy items.


from datetime import datetime, timedelta

date = datetime(2018, 1, 1)
for i in range(10):
    ts.s1.add({'message': 's1-%s' % date}, id=date)
    date += timedelta(days=1)


ts.s1[datetime(2018, 1, 2)::3]

# Returns messages for Jan 2nd - 4th:
[<Message s1 1514872800000-0: {'message': 's1-2018-01-02 00:00:00'}>,
 <Message s1 1514959200000-0: {'message': 's1-2018-01-03 00:00:00'}>,
 <Message s1 1515045600000-0: {'message': 's1-2018-01-04 00:00:00'}>]
 

for message in ts.s1[datetime(2018, 1, 1)::3]:
    print(message.stream, message.timestamp, message.sequence, message.data)

# Prints:
s1 2018-01-01 00:00:00 0 {'message': 's1-2018-01-01 00:00:00'}
s1 2018-01-02 00:00:00 0 {'message': 's1-2018-01-02 00:00:00'}
s1 2018-01-03 00:00:00 0 {'message': 's1-2018-01-03 00:00:00'}
 
```





#### 基本使用


- `XADD`

`XADD mystream * sensor-id 1234 temperature 19.8`

    
    mystream   键名
    
    *  标识Stream中每个条目的条目ID
        （服务器为我们生成新的ID)。每个新的ID都会单调递增，
        
        条目ID: <millisecondsTime>-<sequenceNumber> | 毫秒时间-序列号
        
    sensor-id 1234 
    
    temperature 19.8 
    
         value  键值对
         
 ```python

XADD somestream 0-1 field value
XADD somestream 0-2 foo bar


xrange somestream - + 

>>> 

1) 1) "0-1"
   2) 1) "field"
      2) "value"
2) 1) "0-2"
   2) 1) "field1"
      2) "value2"


# 不能增加ID小于之前的项目 
XADD somestream 0-1 foo bar
(error) ERR The ID specified in XADD is equal or smaller than the target stream top item
```
         

- `XLEN` 

`XLEN  mystream` 查看项目数


- `XRANGE`

 `XRANGE mystream 1518951480106 1518951480107`
 
 `XRANGE somestream 0-1 0-1`
 
 `XRANGE mystream - + COUNT 2`
 
 `XRANGE mystream 1519073279157-1 + COUNT 2`
 
 
- `XREVRANGE` 逆序， 通常用来获取最后的元素
 

 `XREVRANGE mystream + - count 1`
 
 
- `XREAD` 监听到达Stream的新项目
 
 
     订阅到达Stream的新项目
 
 
 
 对比 `发布订阅 pubsub` 或者 `阻塞队列`:
 
     1    Stream 可以有多个客户端（消费者）等待数据, 每个新项目都将传递给等待指定Stream中的数据的每个消费者
          pubsub 每个消费者将获得不同的元素
        
     2   pubsub 中消息是自主引导并且永远不会存储的
                在阻塞列表中，当客户端收到消息时，它会从列表中弹出（有效删除）
                
         Stream  所有消息都无限期地追加在Stream中（除非用户明确要求删除条目）
                 不同的消费者通过记住收到的最后一条消息的ID（last_id），来判断什么是新消息。
                
     3  Streams Consumer Groups(Stream的消费者组)提供 pubsub 无法实现的控制级别 
     
                 已处理项目的明确确认
                 检查待处理项目
                 未处理消息的声明
                 单个客户端的连贯历史可见性。



`xread count 2 STREAMS mystream somestream 0 0`

    订阅多个流
    
    获取大于当前 条目ID的 项目
    
    
`xread block 0 streams mystream $`

    
    获取最新的 项目（多个消费者可以）
    
    相当于 Unix命令tail -f 

 
 

- `XGROUP` 创建 销毁 管理 消费者组


`xgroup CREATE mystream mygroup1 $`

    消费者组将开始消费ID大于您指定的ID的消息
        $ 表示仅消费新消息
        0 表示所有
        

- `XREADGROUP` 消费者组  从Stream中读取


`XREADGROUP GROUP mygroup Alice COUNT 1 STREAMS mystream >`
   
`XREADGROUP GROUP mygroup Alice STREAMS mystream 0` 
    
    特殊ID >    那么该命令将仅返回到目前为止从未传递给其他消费者的 新消息，
                 并且将更新消费者组的最后一个消息ID。
                 
    任何其他有效的数字ID    则该命令将允许我们访问未决消息的   历史记录。
                            也就是说，传递给这个指定消费者的消息集（由提供的名称标识），到目前为止从未用XACK确认过



```python
> XADD mystream * message apple
1526569495631-0
> XADD mystream * message orange
1526569498055-0
> XADD mystream * message strawberry
1526569506935-0
> XADD mystream * message apricot
1526569535168-0
> XADD mystream * message banana
1526569544280-0



XREADGROUP GROUP mygroup Alice COUNT 1 STREAMS mystream >
1) 1) "mystream"
   2) 1) 1) 1526569495631-0
         2) 1) "message"
            2) "apple"


```

- `XACK` 

`XACK mystream mygroup 1526569495631-0`


- `XINFO` 查看stream状态

`xinfo groups stm`

`xinfo CONSUMERS stm cg1`
        
        1) 1) "name"
           2) "cg1"
           3) "consumers"
           4) "1"
           5) "pending"
           6) "1"
           7) "last-delivered-id"
           8) "1594978511726-0"


- `DEL stream_key` 全部删除


- `XTRIM somestream MAXLEN 2` 删除和限制最大条目(最近的两条)


- `XADD somestream maxlen 5 * f1 v2` 保持最近的5条


- `XDEL somestream 1528274353972-0` 删除一个消息


#### 测试实例



```python
# 1 数据准备 xadd

xadd stm * url xiaorui.cc typo 1
xadd stm * url xiaorui.cc typo 2
xadd stm * url xiaorui.cc typo 3
xadd stm * url xiaorui.cc typo 4
xadd stm * url xiaorui.cc typo 5
xadd stm * url xiaorui.cc typo 6
xadd stm * url xiaorui.cc typo 7
xadd stm * url xiaorui.cc typo 8
xadd stm * url xiaorui.cc typo 9
xadd stm * url xiaorui.cc typo 10


# 2 查看 xrange
xrange stm - +
 1) 1) 1528271957975-0
    2) 1) "url"
       2) "xiaorui.cc"
       3) "typo"
       4) "1"
 2) 1) 1528271958716-0
    2) 1) "url"
       2) "xiaorui.cc"
       3) "typo"
       4) "2"
....
....
....


# 3 创建消费组 xgroup create 

xgroup create stm cg1 0-0




# 创建一个消费者 并且消费
xreadgroup GROUP cg1 c1 count 1 streams stm >
1) 1) "stm"
   2) 1) 1) 1528271957975-0
         2) 1) "url"
            2) "xiaorui.cc"
            3) "typo"
            4) "1"


# 使用 xack 来确认信息 

xack stm cg1 1528271957975-0



#  状态已经变成 0  xinfo groups stm

1) 1) "name"
   2) "cg1"
   3) "consumers"
   4) "1"
   5) "pending"
   6) "0"
   7) "last-delivered-id"
   8) "1594978511726-0"


# 多个消费者并发去消费请求

xreadgroup GROUP cg1 c1 COUNT 1 STREAMS stm >

xreadgroup GROUP cg1 c2 COUNT 1 STREAMS stm >


#  >   表示从当前消费组的last_delivered_id后面开始读
# 每当消费者读取一条消息，last_delivered_id变量就会前进

# get消息和set偏移量的过程是原子的


```


#### 实际应用



[redis-streams-with-python](http://charlesleifer.com/blog/redis-streams-with-python/)



