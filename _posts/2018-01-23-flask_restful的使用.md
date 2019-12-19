---
title: flask_restful的使用
description: flask_restful的使用
categories:
- python
tags:
- flask
---

<br>


# flask_restful的使用

<br>

`基本demo:`

```python

    from flask import Flask
    from flask_restful import Resource, Api
    
    app = Flask(__name__)
    api = Api(app)
    
    class HelloWorld(Resource):
        def get(self):
            return {'hello': 'world'}
    
    api.add_resource(HelloWorld, '/')
    
    if __name__ == '__main__':
        app.run(debug=True)
        
    
    #  curl http://127.0.0.1:5000/  --->> json格式 {"hello": "world"}
```