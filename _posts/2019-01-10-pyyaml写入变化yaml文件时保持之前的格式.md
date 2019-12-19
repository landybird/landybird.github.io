---
title: pyyaml写入变化yaml文件时保持之前的格式
description: pyyaml写入变化yaml文件时保持之前的格式
categories:
- python
tags:
- python
---

<br>


# pyyaml写入变化yaml文件时保持之前的格式


<br>


`yaml 文件`

```yaml

test:
  age: 12
  name: MAX
  version: '2.2'


```

python `yaml.dump()`写入处理时，指定`default_flow_style=False `

```python

import yaml

with open("demo.yaml", "r") as yaml_file:
    yaml_dic = yaml.load(yaml_file.read())
    test_dict = yaml_dic["test"]
    print(yaml_dic)

with open("demo.yaml", "w+") as yaml_file:
    test_dict["name"] = "MIN"
    test_dict["age"] = 19
    test_dict["version"] = "2.1"
    yaml.dump(yaml_dic, yaml_file, default_flow_style=False)

with open("demo.yaml", "r") as yaml_file:
    yaml_dic = yaml.load(yaml_file.read())
    test_dict = yaml_dic["test"]
    print(yaml_dic)


```

变换后的yaml 文件

```yaml
test:
  age: 19
  name: MIN
  version: '2.1'


```


