---
title: python自动生成command命令--python_fire包
description: python自动生成command命令--python_fire包
categories:
- python
tags:
- python基础
---

<br>


# python自动生成command命令--python_fire

<br>

## 1 下载安装  `Installation`
    
    
    pip install fire
    
    
<br>

## 2 使用 Demo 


```python
     
    import fire
    
    class Calculate(object):
    
        def double(self, number):
            return 2 * number
    
    
    if __name__ == '__main__':
        fire.Fire(Calculate)
    
    
    类似于Yii中的Command类
    
    >>  python filename.py  类名(命令名)  函数名(任务名)  --param=xxx(函数的参数)
    
    
```
    
   
<br>

## 3 详细使用方法(不同的version)

> `fire.Fire()`


```python

    import fire
    
    def hello(name):
      return 'Hello {name}!'.format(name=name)
    
    if __name__ == '__main__':
      fire.Fire()
      
      
    python example.py hello World -- 函数 + 参数

```


> `fire.Fire(<fn>)`

```python

    import fire
    
    def hello(name):
      return 'Hello {name}!'.format(name=name)
    
    if __name__ == '__main__':
      fire.Fire(hello)

    
    python example.py World  参数

```

> fire.Fire(<cls>)

```python

    
    import fire
    
    class Calculate(object):
    
        def double(self, number):
            return 2 * number
    
    
    if __name__ == '__main__':
        fire.Fire(Calculate)

    python example.py  double --number=1
    
```


    
