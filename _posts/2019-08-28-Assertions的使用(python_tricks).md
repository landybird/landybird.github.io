---
title: Assertions的使用
description: Assertions的使用
categories:
- python
tags:
- python tricks
---

<br>

#### python的断言--Assertions


- 什么是 Assertions

使用assertions的意义： 在于 `测试某种条件的 debugging aid` 

    如果条件是 true, 程序正常进行，否则 AssertionError会抛出 
   
先来看一个例子


```python

def apply_discount(product, discount):
    price = int(product['price'] * (1.0 - discount))
    assert 0 <= price <= product['price']
    return price

shoes = {'name': 'Fancy Shoes', 'price': 14900}

apply_discount(shoes, 0.25)
>> 11175

apply_discount(shoes, 2.0)

Traceback (most recent call last):
    File "<input>", line 1, in <module>
        apply_discount(prod, 2.0)
    File "<input>", line 4, in apply_discount
        assert 0 <= price <= product['price']
AssertionError

```

Assertion可以根据exception stacktrace很好的定位出现问题的条件

- 为什么不使用常规的 exception?

恰当的使用 assertions 会通知开发不可恢复的错误， 而不是像 `Fle-Not-Found` 一样表明一个条件信号， 然后开发者可以针对性的重试

要明确 assertions 的目的 是用来 快速debugging


- python中 Assert的语法结构

`assert_stmt ::= "assert" expression1 ["," expression2]`


这里的 expression1 是 测试的条件表达式, expression2 是 error_message
```python

    assert False, (
        'This should never happen, but it does '
        'occasionally. We are currently trying to '
        'figure out why. Email dbader if you '
        'encounter this in the wild. Thanks!')
```

- 使用Assertions的两点警告


<1> 不要用来做 数据条件的验证

python中的 assertions 可以使用 `-O`或者 `-OO` 参数 全局的禁用
设置全局的环境变量 `PYTHONOPTIMIZE`也是同样的效果

[PYTHONOPTIMIZE](https://docs.python.org/3/using/cmdline.html#envvar-PYTHONOPTIMIZE)


在禁用的情况下, assertions 会pass掉， 永远不会执行

```python
# 如果 使用全局变量 -O ， 判断会直接pass， 进行后面的操作

def delete_product(prod_id, user):
    assert user.is_admin(), 'Must be admin'
    assert store.has_product(prod_id), 'Unknown product'
    store.get_product(prod_id).delete()

# 应该使用 regular exception

def delete_product(product_id, user):
    if not user.is_admin():
        raise AuthError('Must be admin to delete')
    if not store.has_product(product_id):
        raise ValueError('Unknown product id')
    store.get_product(product_id).delete()
```


<2> 注意 assert 的结构， 后面不是元祖 `()`

`assert 1 == 2, 'This should fail'`

注意区别于

`assert(1 == 2, 'This should fail')` 永远都是`True`
