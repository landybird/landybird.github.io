---
title: 工厂模式(learning_python_design_patterns)
description: 工厂模式(learning_python_design_patterns)
categories:
- python
tags:
- learning python design patterns
---

<br>

![](https://landybird.github.io/landybird.github.io/assets/images/lpdp2.png)


一个简单的例子


```python

# 这里的 factory 有一个 静态方法 build_connection

class SimpleFactory(object):
    @staticmethod # This decorator allows to run method withoutclass instance, .e. SimpleFactory.build_connection
    def build_connection(protocol):
        if protocol == 'http':
            return HTTPConnection()
        elif protocol == 'ftp':
            return FTPConnection()
        else:
            raise RuntimeError('Unknown protocol')
            
if __name__ == '__main__':
    protocol = raw_input('Which Protocol to use? (http or ftp): ')
    protocol = SimpleFactory.build_connection(protocol)
    protocol.connect()
    print(protocol.get_response())

```

![](https://landybird.github.io/landybird.github.io/assets/images/lpdp3.png)


```python

import abc
import urllib2
from BeautifulSoup import BeautifulStoneSoup

class Connector(metaclass=abc.ABCMeta):

    def __init__(self, is_secure):
        self.is_secure = is_secure
        self.port = self.port_factory_method()
        self.protocol = self.protocol_factory_method()

    @abc.abstractmethod
    def parse(self, content):
        pass

    def read(self, host, path):
        url = self.protocol + '://' + host + ':' + str(self.port) + path
        print('Connecting to ', url)
        return urllib2.urlopen(url, timeout=2).read()

    @abc.abstractmethod
    def protocol_factory_method(self):
        pass

    @abc.abstractmethod
    def port_factory_method(self):
        return FTPPort()



class HTTPConnector(Connector):

    def protocol_factory_method(self):
        if self.is_secure:
            return 'https'
        return 'http'

    def port_factory_method(self):
        """Here HTTPPort and HTTPSecurePort are concrete objects,
        created by factory method."""
        if self.is_secure:
            return HTTPSecurePort()
        return HTTPPort()

    def parse(self, content):
        """Parses web content."""
        filenames = []
        soup = BeautifulStoneSoup(content)
        links = soup.table.findAll('a')
        for link in links:
            filenames.append(link['href'])
        return '\n'.join(filenames)


class FTPConnector(Connector):
    """A concrete creator that creates a FTP connector and sets in
    runtime all its attributes."""
    def protocol_factory_method(self):
        return 'ftp'

    def port_factory_method(self):
        return FTPPort()

    def parse(self, content):
        lines = content.split('\n')
        filenames = []
        for line in lines:
            # The FTP format typically has 8 columns, split them
            splitted_line = line.split(None, 8)
            if len(splitted_line) == 9:
                filenames.append(splitted_line[-1])
        return '\n'.join(filenames)



class Port(object):
    __metaclass__ = abc.ABCMeta

    @abc.abstractmethod
    def __str__(self):
        pass

class HTTPPort(Port):
    """A concrete product which represents http port."""
    def __str__(self):
        return '80'

class HTTPSecurePort(Port):
    """A concrete product which represents https port."""
    def __str__(self):
        return '443'

class FTPPort(Port):
    """A concrete product which represents ftp port."""
    def __str__(self):
        return '21'




if __name__ == '__main__':
    domain = 'ftp.freebsd.org'
    path = '/pub/FreeBSD/'
    protocol = input('Connecting to {}. Which Protocol to use? (0-http,1-ftp): '.format(domain))
    if protocol == 0:
        is_secure = bool(input('Use secure connection? (1-yes, 0-no): '))
        connector = HTTPConnector(is_secure)
    else:
        is_secure = False
        connector = FTPConnector(is_secure)
    try:
        content = connector.read(domain, path)
    except urllib2.URLError as  e:
        print('Can not access resource with this method')
    else:
        print(connector.parse(content))


```
