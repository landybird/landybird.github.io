---
title: 小结一些markdown的语法
categories:
- markdown
tags:
- markdown
---

<br>

**总结一些常用的markdown语法.**

# 标题

	# Heading 1
	
	## Heading 2
	
	### Heading 3
	
	#### Heading 4
	
	##### Heading 5
	
	###### Heading 6



# 文本


>1 [连接](url)

	[连接](url)

>2 **粗体**

	**Strong text** 

>3 *斜体*

	*Italic text*

>4 ~~删除内容~~ 

	~~Deleted text~~


>5`标签`

	`tags`

>6 引文

	>引文


# 表格，列表



> 自定义列表 (dl)

<dl><dt>标题</dt><dd>内容</dd></dl>

	<dl><dt>Definition List Title</dt>
	<dd>This is a definition list division.</dd></dl>

> Ordered List (ol)

1. List Item 1
2. List Item 2
3. List Item 3

	
		1. List Item 1
		2. List Item 2
		3. List Item 3

> Unordered List (ul)

- List Item 1
- List Item 2
- List Item 3

	- List Item 1
	- List Item 2
	- List Item 3
			
			- List Item 1
			- List Item 2
			- List Item 3

> 表格

| Table Header 1 | Table Header 2 | Table Header 3 |
| --- | --- | --- |
| Division 1 | Division 2 | Division 3 |
| Division 1 | Division 2 | Division 3 |
| Division 1 | Division 2 | Division 3 |

	| Table Header 1 | Table Header 2 | Table Header 3 |
	| --- | --- | --- |
	| Division 1 | Division 2 | Division 3 |
	| Division 1 | Division 2 | Division 3 |
	| Division 1 | Division 2 | Division 3 |

# 图片
![](http://ww3.sinaimg.cn/mw690/81b78497jw1emfgwjrh2pj21hc0u01g3.jpg)
	
	![](http://ww2.sinaimg.cn/mw690/81b78497jw1emfgwil5xkj21hc0u0tpm.jpg)

# 表情

 :smile: 笑脸

	:heart_eyes::kissing_heart::kissing_closed_eyes::flushed::relieved::satisfied::grin:

# code

```python
import re
from django.http import Response
print(re.findall(r'',''))
```

	```python
	import re
	from django.http import Response
	print(re.findall(r'',''))
	```

