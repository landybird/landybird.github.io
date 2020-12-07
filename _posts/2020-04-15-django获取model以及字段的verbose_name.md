---
title: django获取model以及字段的verbose_name                               
description: django获取model以及字段的verbose_name
categories:
- python
tags:
- django   
---

```python
# model中的字段verbose_name  
model._meta.get_field("field_name").verbose_name


# model的verbose_name
model._meta.verbose_name
```