---
title: requests 和 aiohttp 中解决ssl错误
description: requests 和 aiohttp 中解决ssl错误
categories:
- python
tags:
- python基础
---


<br>

# `requests`和`aiohttp`中解决ssl错误

<br>

某些网站对 ssl 1.0 以上的版本支持并不友好，但是 python ssl lib 默认会指定当前OpenSSL支持的最高版本，并且当服务器返回的ssl版本号低于或者等于1.0时, 会出现如下错误


aiohttp

    aiohttp.client_exceptions.ClientConnectorError: Cannot connect to host facebook.com:443 ssl:[None]
    
requests

     Caused by SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:645)SSLErrorEOF occurred in violation of protocol 


查看当前client的ssl版本

```python
python -c "import ssl; print(ssl.OPENSSL_VERSION)"
```

查看本地 openssl version

    openssl version
    

查看通讯过程使用的额ssl


    OpenSSL> s_client  -connect www.facebook.com:443 -msg
   
    
    SSL-Session:
        Protocol  : TLSv1.2
        Cipher    : ECDHE-ECDSA-AES128-GCM-SHA256
        Session-ID: 3D78B4E33A5376E697C95E9ADD1E4BFDBB69A564EDE7F262E5D4A6D6D96AAE2B
        Session-ID-ctx:
        Master-Key: 5F8B981090E231DEF760DC0A1F5DCBBB258B62A288A5706FBE86719BAD6E25ED4F68B0F362FDDDD095AF67CA3754E105
        Key-Arg   : None
        Krb5 Principal: None
        PSK identity: None
        PSK identity hint: None
        TLS session ticket lifetime hint: 172800 (seconds)
        TLS session ticket:
        0000 - 7f 42 fc d9 b1 f2 9d 96-6d ef a4 45 cb 27 c7 95   .B......m..E.'..
        0010 - fc 91 58 79 77 4b ef 3b-a3 0b 43 14 3e 51 83 27   ..XywK.;..C.>Q.'
        0020 - a5 0a 53 a6 1f 06 d3 cd-d6 b3 0d 4f a0 5f a0 bc   ..S........O._..
        0030 - 5b f8 25 5e 69 77 df 75-7f fc 76 44 eb 5d 26 6d   [.%^iw.u..vD.]&m
        0040 - cc e7 0b 84 b2 6e 4d 6c-f4 33 89 55 a5 69 58 fc   .....nMl.3.U.iX.
        0050 - bd c1 51 8d 70 2c 52 47-c5 31 b3 66 47 4e 13 18   ..Q.p,RG.1.fGN..
        0060 - 65 4a f9 48 71 34 50 12-7f 9a 6c 8d 01 11 9b 25   eJ.Hq4P...l....%
        0070 - e9 42 b4 61 59 60 0f 0e-59 38 54 1c dd 32 f6 ce   .B.aY`..Y8T..2..
        0080 - 69 b9 be a2 be 17 5a e3-69 f2 db ab b2 2f ca 3e   i.....Z.i..../.>
        0090 - 79 87 2d a0 e2 d7 93 15-77 34 86 53 3d c4 13 8a   y.-.....w4.S=...
        00a0 - 12 73 59 22 24 d7 49 84-63 4d 5d 9b 2f b7 12 26   .sY"$.I.cM]./..&
    
        Start Time: 1556282230
        Timeout   : 300 (sec)
        Verify return code: 0 (ok)
    ---
    <<< TLS 1.2 Alert [length 0002], warning close_notify
        01 00
    closed
    >>> TLS 1.2 Alert [length 0002], warning close_notify
        01 00



curl 和 aiohttp , requests等其他库默认使用安全的 ssl/tls 版本

    curl is designed to use a "safe version" of SSL/TLS by default. It means that it will not negotiate SSLv2 or SSLv3 unless specifically told to,
    and in fact several TLS libraries no longer provide support for those protocols so in many cases curl is not even able to speak those protocol versions 
    unless you make a serious effort.
    


修改ssl_version

```python


class MyAdapter(HTTPAdapter):
    # 使用  TLSv1  Caused by SSLError(SSLEOFError(8, 'EOF occurred in violation of protocol (_ssl.c:645)SSLErrorEOF occurred in violation of protocol
    def init_poolmanager(self, connections, maxsize, block=False, **pool_kwargs):
        self.poolmanager = PoolManager(
            num_pools=connections,
            maxsize=maxsize,
            block=block,
            ssl_version=ssl.PROTOCOL_TLSv1
        )

    #  s=requests.session()
    # s.mount("https://", MyAdapter)



class SSLAdapter(HTTPAdapter):
    '''An HTTPS Transport Adapter that uses an arbitrary SSL version.'''
    def __init__(self, ssl_version=None, **kwargs):
        self.ssl_version = ssl_version

        super(SSLAdapter, self).__init__(**kwargs)

    def init_poolmanager(self, connections, maxsize, block=False, **pool_kwargs):
        self.poolmanager = PoolManager(num_pools=connections,
                                       maxsize=maxsize,
                                       block=block,
                                       ssl_version=self.ssl_version)

```

```python

import ssl
import aiohttp
import asyncio
 
 
async def test():
    FORCED_CIPHERS = (
        'ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+HIGH:'
        'DH+HIGH:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+HIGH:RSA+3DES'
    )
    sslcontext = ssl.create_default_context()
    sslcontext.options |= ssl.OP_NO_SSLv3
    sslcontext.options |= ssl.OP_NO_SSLv2
    sslcontext.options |= ssl.OP_NO_TLSv1_1
    sslcontext.options |= ssl.OP_NO_TLSv1_2
    sslcontext.options |= ssl.OP_NO_TLSv1_3
    sslcontext.set_ciphers(FORCED_CIPHERS)
 
    async with aiohttp.ClientSession(connector=aiohttp.TCPConnector(limit=50, loop=loop)) as session:
        r = await session.get('https://www.mdnkids.com/news/?Serial_NO=108552', ssl=sslcontext)
        print(await r.text())
 
if __name__ == "__main__":
    loop = asyncio.get_event_loop()
    loop.run_until_complete(test())


```




