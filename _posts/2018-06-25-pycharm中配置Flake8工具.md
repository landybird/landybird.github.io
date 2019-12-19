---
title: pycharm中配置Flake8工具与autopep8
description: pycharm中配置Flake8工具
categories:
- python
tags:
- python基础
---

<br>


#  pycharm中配置Flake8工具

1 `File`->`Setting`->`Tools`->`external tools`->添加工具


2 配置参数：

    Program: $PyInterpreterDirectory$/python  
           // 配置下的python解释器的python目录
    Arguments: -m flake8 --show-source --statistics $ProjectFileDir$  
          // 参数
    Working directory: $ProjectFileDir$  
         // 当前的项目路径
    Output Filter: (留空就可以了, pycharm能自动识别路径.)
    
    
3 在 Tools工具栏 -> external tools 中使用


    结果如下：
          ^
    C:\Users\Domob\Desktop\dev\smb_middle_server\user_account\models.py:16:1: W391 blank line at end of file
    
    ^
    C:\Users\Domob\Desktop\dev\smb_middle_server\user_account\tests.py:1:1: F401 'django.test.TestCase' imported but unused
    from django.test import TestCase
    ^
    C:\Users\Domob\Desktop\dev\smb_middle_server\user_account\views.py:1:1: F401 'django.shortcuts.render' imported but unused
    from django.shortcuts import render
    ^
    ...
    
    
    C:\Users\Domob\Desktop\dev\smb_middle_server\user_account\__init__.py:1:59: W292 no newline at end of file
    default_app_config = 'user_account.apps.UserAccountConfig'                                                          ^
    2     E101 indentation contains mixed spaces and tabs
    2     E114 indentation is not a multiple of four (comment)
    2     E116 unexpected indentation (comment)
    16    E122 continuation line missing indentation or outdented
    1     E128 continuation line under-indented for visual indent
    2     E131 continuation line unaligned for hanging indent
    4     E201 whitespace after '['
    6     E202 whitespace before ']'
    2     E203 whitespace before ':'
    
    ... 
    
    


<br>


#  pycharm中配置autoPEP8

1 `File`->`Setting`->`Tools`->`external tools`->添加工具


2 配置参数：

    Program: $PyInterpreterDirectory$/autopep8  
           // 配置下的python解释器的python目录
    Arguments: --in-place --aggressive --aggressive $FilePath$
          // 参数
    Working directory: $ProjectFileDir$  
         // 当前的项目路径
    Output Filter: $FILE_PATH$:$LINE$:$COLUMN$:.*
    
    
3 在 Tools工具栏 -> external tools 中使用 `自动调整代码格式`

    