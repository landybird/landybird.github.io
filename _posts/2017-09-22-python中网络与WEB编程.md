---
title: python中网络与WEB编程
description: python中网络与WEB编程
categories:
- python
tags:
- python基础
---

<br>


# python中网络与WEB编程

<br>

## TCP
 
实现一个`服务器`，通过`TCP协议`和`客户端通信` 使用 `socketserver库` 

```python


服务端
    
    from socketserver import BaseRequestHandler, TCPServer
    
    class EchoHandler(BaseRequestHandler):
        def handle(self):
            print('Got connection from', self.client_address)
            while True:
    
                msg = self.request.recv(8192)
                if not msg:
                    break
                self.request.send(msg)
    
    if __name__ == '__main__':
        serv = TCPServer(('', 20000), EchoHandler)
        serv.serve_forever()



客户端
    >>> from socket import socket, AF_INET, SOCK_STREAM
    >>> s = socket(AF_INET, SOCK_STREAM)
    >>> s.connect(('localhost', 20000))
    >>> s.send(b'Hello')
    5
    >>> s.recv(8192)
    b'Hello'
>>>

```



```python

一个服务器简单例子:
    
    from socket import socket, AF_INET, SOCK_STREAM
    
    def echo_handler(address, client_sock):
        print('Got connection from {}'.format(address))
        while True:
            msg = client_sock.recv(8192)
            if not msg:
                break
            client_sock.sendall(msg)
        client_sock.close()
    
    def echo_server(address, backlog=5):
        sock = socket(AF_INET, SOCK_STREAM)
        sock.bind(address)
        sock.listen(backlog)
        while True:
            client_sock, client_addr = sock.accept()
            echo_handler(client_addr, client_sock)
    
    if __name__ == '__main__':
        echo_server(('', 20000))


```


```python


    socket文件接口
    
        from socketserver import StreamRequestHandler, TCPServer
        
        class EchoHandler(StreamRequestHandler):
            def handle(self):
                print('Got connection from', self.client_address)
                # self.rfile is a file-like object for reading
                for line in self.rfile:
                    # self.wfile is a file-like object for writing
                    self.wfile.write(line)
        
        if __name__ == '__main__':
            serv = TCPServer(('', 20000), EchoHandler)
            serv.serve_forever()


```



> 注意 默认情况下这种服务器是`单线程的`，`一次只能为一个客户端连接服务`

使用` ForkingTCPServer `和 `ThreadingTCPServer `可以创建多线程服务

使用fork或线程服务器  `潜在问题` 就是它们会为每个客户端连接创建一个新的进程或线程, 
`恶意的黑客`可以`同时发送大量的连接`让你的`服务器奔溃`

这时候应该使用`线程池`


<br>

## UDP 


由于`没有底层的连接`，UPD服务器相对于TCP服务器来讲实现起来更加简单。 

不过，UDP天生是`不可靠`的（因为`通信没有建立连接，消息可能丢失`）。 因此需要由你自己来决定该怎样处理丢失消息的情况。

 UDP通常被用在那些`对于可靠传输要求不是很高的场合`。
 
```python

from socket import socket, AF_INET, SOCK_DGRAM
import time

def time_server(address):
    sock = socket(AF_INET, SOCK_DGRAM)
    sock.bind(address)
    while True:
        msg, addr = sock.recvfrom(8192)
        print('Got message from', addr)
        resp = time.ctime()
        sock.sendto(resp.encode('ascii'), addr)

if __name__ == '__main__':
    time_server(('', 20000))

```

<br>

## web编程，实现一个微型的框架(`简单的服务器`)

```python

testcgi.py
     
        # 基于WSGI标准
        
        import cgi
        
        def notfound_404(environ, start_response):
            start_response('404 Not Found', [ ('Content-type', 'text/plain') ])
            return [b'Not Found']
        
        class PathDispatcher:
            def __init__(self):
                self.pathmap = {}
        
            def __call__(self, environ, start_response):
                # 根据传递的 请求 和 路由 ，映射到视图函数处理
                
                path = environ['PATH_INFO']
                params = cgi.FieldStorage(environ['wsgi.input'],
                                          environ=environ)
                method = environ['REQUEST_METHOD'].lower()
                environ['params'] = { key: params.getvalue(key) for key in params }
                handler = self.pathmap.get((method,path), notfound_404)
                return handler(environ, start_response)
        
            def register(self, method, path, function):
                # 绑定path和method，以及视图函数的关系 
                self.pathmap[method.lower(), path] = function
                return function


server.py  启动服务端，使用上面的 testcgi 模块
           
        import time
        
        _hello_resp = '''\
        <html>
          <head>
             <title>Hello {name}</title>
           </head>
           <body>
             <h1>Hello {name}!</h1>
           </body>
        </html>'''
        
        def hello_world(environ, start_response):
            start_response('200 OK', [ ('Content-type','text/html')])
            params = environ['params']
            resp = _hello_resp.format(name=params.get('name'))
            yield resp.encode('utf-8')
        
        _localtime_resp = '''\
        <?xml version="1.0"?>
        <time>
          <year>{t.tm_year}</year>
          <month>{t.tm_mon}</month>
          <day>{t.tm_mday}</day>
          <hour>{t.tm_hour}</hour>
          <minute>{t.tm_min}</minute>
          <second>{t.tm_sec}</second>
        </time>'''
        
        def localtime(environ, start_response):
            start_response('200 OK', [ ('Content-type', 'application/xml') ])
            resp = _localtime_resp.format(t=time.localtime())
            yield resp.encode('utf-8')
        
        if __name__ == '__main__':
            from wsgiref.simple_server import make_server
            from test import PathDispatcher
        
            # Create the dispatcher and register functions
            dispatcher = PathDispatcher()
            dispatcher.register('GET', '/hello', hello_world)
            dispatcher.register('GET', '/localtime', localtime)
        
            # Launch a basic server
            httpd = make_server('', 8080, dispatcher)
            print('Serving on port 8080...')
            httpd.serve_forever()


客户端 访问服务端

    >>> u = urlopen('http://localhost:8080/hello?name=Guido')
        >>> print(u.read().decode('utf-8'))
        <html>
          <head>
             <title>Hello Guido</title>
           </head>
           <body>
             <h1>Hello Guido!</h1>
           </body>
        </html>
    
    >>> u = urlopen('http://localhost:8080/localtime')
        >>> print(u.read().decode('utf-8'))
        <?xml version="1.0"?>
        <time>
          <year>2012</year>
          <month>11</month>
          <day>24</day>
          <hour>14</hour>
          <minute>49</minute>
          <second>17</second>
        </time>
        >>>

```

<br>

## `XML-RPC`实现`简单的远程调用`


```python

服务端
    
    from xmlrpc.server import SimpleXMLRPCServer
    
    class KeyValueServer:
        _rpc_methods_ = ['get', 'set', 'delete', 'exists', 'keys']
        def __init__(self, address):
            self._data = {}
            self._serv = SimpleXMLRPCServer(address, allow_none=True)
            for name in self._rpc_methods_:
                self._serv.register_function(getattr(self, name))
    
        def get(self, name):
            return self._data[name]
    
        def set(self, name, value):
            self._data[name] = value
    
        def delete(self, name):
            del self._data[name]
    
        def exists(self, name):
            return name in self._data
    
        def keys(self):
            return list(self._data)
    
        def serve_forever(self):
            self._serv.serve_forever()
    
    # Example
    if __name__ == '__main__':
        kvserv = KeyValueServer(('', 15000))
        kvserv.serve_forever()



客户端

    >>> from xmlrpc.client import ServerProxy
    >>> s = ServerProxy('http://localhost:15000', allow_none=True)
    >>> s.set('foo', 'bar')
    >>> s.set('spam', [1, 2, 3])
    >>> s.keys()
    ['spam', 'foo']
    >>> s.get('foo')
    'bar'
    >>> s.get('spam')
    [1, 2, 3]
    >>> s.delete('spam')
    >>> s.exists('spam')
    False
    >>>
    

```

XML-RPC 可以很容易的构造一个简单的`远程调用服务`。

    创建一个服务器实例， 通过它的方法 register_function() 来注册函数，
    然后使用方法 serve_forever() 启动它。 

XML-RPC暴露出来的函数`只能适用于部分数据类型`，比如字符串、整形、列表和字典。 

SimpleXMLRPCServer 的实现是`单线程`的, 性能是缺点


<br>

## python 解释器间的交互 `multiprocessing.connection`




```python

客户端与服务端的交互
    
    from multiprocessing.connection import Listener
    import traceback
    
    def echo_client(conn):
        try:
            while True:
                msg = conn.recv()
                conn.send(msg)
        except EOFError:
            print('Connection closed')
    
    def echo_server(address, authkey):
        serv = Listener(address, authkey=authkey)
        while True:
            try:
                client = serv.accept()
                print(client.recv())
                echo_client(client)
            except Exception:
                traceback.print_exc()
    
    echo_server(('', 25000), authkey=b'peekaboo')

    发送信息测试：
        
        >>> from multiprocessing.connection import Client
        >>> c = Client(('localhost', 25000), authkey=b'peekaboo')
        >>> c.send('hello')
        >>> c.recv()
        'hello'
        >>> c.send(42)
        >>> c.recv()
        42
        >>> c.send([1, 2, 3, 4, 5])
        >>> c.recv()
        [1, 2, 3, 4, 5]
        >>>
```



跟`底层socket` 不同的:

每个`消息会完整保存`（每一个通过send()发送的对象能通过recv()来完整接受）。 

另外，所有对象会`通过pickle序列化`。

因此，`任何兼容pickle的对象都能在此连接上面被发送和接受`。

<br>

## 简单的客户端认证  ` hmac 模块` 

实现一个连接握手，从而实现一个简单而高效的认证过程

`hmac认证算法基于哈希函数如MD5和SHA-1`


原理：

基本原理是当`连接建立后`，`服务器给客户端发送一个随机的字节消息`（这里例子中使用了 os.urandom() 返回值）。
客户端和服务器`同时利用hmac`和`一个只有双方知道的密钥`来计算出一个加密哈希值。

然后客户端将它计算出的摘要发送给服务器，服务器通过比较这个值和自己计算的是否一致来决定接受或拒绝连接。
`摘要的比较` 需要使用` hmac.compare_digest() 函数`。 使用这个函数可以避免遭到时间分析攻击，不要用简单的比较操作符（==）。 

为了使用这些函数，你需要将它集成到已有的网络或消息代码中。

```python

import hmac
import os

def client_authenticate(connection, secret_key): 
    
    # 客户端加密
    '''
    Authenticate client to a remote service.
    connection represents a network connection.
    secret_key is a key known only to both client/server.
    '''
    message = connection.recv(32)
    hash = hmac.new(secret_key, message)
    digest = hash.digest()
    connection.send(digest)

def server_authenticate(connection, secret_key):

    # 服务端加密 验证
    '''
    Request client authentication.
    '''
    message = os.urandom(32)
    connection.send(message)
    hash = hmac.new(secret_key, message)
    digest = hash.digest()
    response = connection.recv(len(digest))
    return hmac.compare_digest(digest,response)


```

```python

网络通讯中使用加密验证
    
    
    服务端：
        
        from socket import socket, AF_INET, SOCK_STREAM
        
        secret_key = b'peekaboo'
        def echo_handler(client_sock):
            if not server_authenticate(client_sock, secret_key):
                # 验证
                client_sock.close()
                return
            while True:
        
                msg = client_sock.recv(8192)
                if not msg:
                    break
                client_sock.sendall(msg)
        
        def echo_server(address):
            s = socket(AF_INET, SOCK_STREAM)
            s.bind(address)
            s.listen(5)
            while True:
                c,a = s.accept()
                echo_handler(c)
        
        echo_server(('', 18000))
        
    
    客户端:
        
        from socket import socket, AF_INET, SOCK_STREAM
        
        secret_key = b'peekaboo'
        
        s = socket(AF_INET, SOCK_STREAM)
        s.connect(('localhost', 18000))
        client_authenticate(s, secret_key)
        # 发送验证 secret_key
        s.send(b'Hello World')
        resp = s.recv(1024)

```


<br>

## 网络服务中加入SSL

`ssl 模块` 能为底层socket连接添加SSL的支持。

`ssl.wrap_socket() 函数` 接受一个已存在的`socket`作为参数并`使用SSL层来包装它`。


```python

服务端：

    from socket import socket, AF_INET, SOCK_STREAM
    import ssl
    
    KEYFILE = 'server_key.pem'   # Private key of the server
    CERTFILE = 'server_cert.pem' # Server certificate (given to client)
    
    def echo_client(s):
        while True:
            data = s.recv(8192)
            if data == b'':
                break
            s.send(data)
        s.close()
        print('Connection closed')
    
    def echo_server(address):
        s = socket(AF_INET, SOCK_STREAM)
        s.bind(address)
        s.listen(1)
    
        # Wrap with an SSL layer requiring client certs
        s_ssl = ssl.wrap_socket(s,
                                keyfile=KEYFILE,
                                certfile=CERTFILE,
                                server_side=True
                                )
        # Wait for connections
        while True:
            try:
                c,a = s_ssl.accept()
                print('Got connection', c, a)
                echo_client(c)
            except Exception as e:
                print('{}: {}'.format(e.__class__.__name__, e))
    
    echo_server(('', 20000))


客户端：
    
    >>> from socket import socket, AF_INET, SOCK_STREAM
    >>> import ssl
    >>> s = socket(AF_INET, SOCK_STREAM)
    >>> s_ssl = ssl.wrap_socket(s,
                    cert_reqs=ssl.CERT_REQUIRED,
                    ca_certs = 'server_cert.pem')
    >>> s_ssl.connect(('localhost', 20000))
    >>> s_ssl.send(b'Hello World?')
    12
    >>> s_ssl.recv(8192)
    b'Hello World?'

```

<br>

## 网络连接发送和接受连续数据的大型数组，并尽量减少数据的复制操作 `memoryview`函数


`bytearray` 是可变(mutable)的字节序列，相对于Python2中的str，但str是不可变(immutable)的。
在`Python3`中由于str默认是unicode编码，所以`只有通过bytearray才能按字节访问`。

`str`和`bytearray`的`切片操作`会产生新的切片str和bytearry并拷贝数据，使用`memoryview不会`。


```python

    不使用memoryview
        
        >> a = 'aaaaaa'
        >> b = a[:2]    # 会产生新的字符串
        
        >> a = bytearray('aaaaaa')
        >> b = a[:2]    # 会产生新的bytearray
        >> b[:2] = 'bb' # 对b的改动不影响a
        >> a
        bytearray(b'aaaaaa')
        >> b
        bytearray(b'bb')
        
        
    使用memoryview
    
        >> a = 'aaaaaa'
        >> ma = memoryview(a)
        >> ma.readonly  # 只读的memoryview
        True
        >> mb = ma[:2]  # 不会产生新的字符串
        
        >> a = bytearray('aaaaaa')
        >> ma = memoryview(a)
        >> ma.readonly  # 可写的memoryview
        False
        >> mb = ma[:2]      # 不会会产生新的bytearray
        >> mb[:2] = 'bb'    # 对mb的改动就是对ma的改动
        >> mb.tobytes()
        'bb'
        >> ma.tobytes()
        'bbaaaa'

```

网络连接发送和接受连续数据的大型数组，并尽量减少数据的复制操作 

```python
    
    def read(size):
        ret = '' 
        remain = size
        while True:
            data = sock.recv(remain)
            ret += data     # 这里不断会有新的str对象产生
            if len(data) == remain:
                break
            remain -= len(data)
        return ret


使用meoryview之后，避免了不断的字符串拼接和新对象的产生
    
    def read(size):
        ret = memoryview(bytearray(size)) 
        remain = size
        while True:
            data = sock.recv(remain)
            length = len(data)
            ret[size - remain: size - remain + length] = data
            if len(data) == remain:
                break
            remain -= len(data)
        return ret


返回memoryview还有一个优点，在使用struct进行unpack解析时可以直接接收memoryview对象，非常高效（避免大的str进行分段解析时大量的切片操作）。

    例如：spe, len = struct.unpack('!BI', mv[:5])
```

