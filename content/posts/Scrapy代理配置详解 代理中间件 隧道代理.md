+++
title = 'Scrapy代理配置详解'
date = 2024-02-19T14:32:42+08:00
draft = false
description = "Scrapy代理配置使用记录，包括各种形式代理基础使用和代理中间件示例;解决了Scrapy使用代理隧道验证不成功的问题" 
tags = [ "Scrapy", "Python",] 
categories = [ "Python"]
+++
# Scrapy代理配置使用记录

废话不说，直接上重点，有哪些代理配置方式？

- 在spider代码中配置；这种方式极不推荐，虽然可以实现，但是spider中就该写采集逻辑，不应该将代理配置加入其中。
- 配置代理中间件；推荐的方式，中间件是可插拔式的，使用和配置也很简单。

[TOC]

---

## 在spider中指定代理

这种方式虽然我极不推荐，但是也简单介绍下。

在你的爬虫代码中，在`Request`方法的`meta`参数中加上`proxy`字段即可，这样就可以给这个request配置一个静态的IP代理。

```python
# -*- coding: utf-8 -*-
import scrapy


class TestSpider(scrapy.Spider):
    name = 'test'
    allowed_domains = ['test.com']
    start_urls = ['http://httpbin.org/ip']

    def start_requests(self):  # 控制爬虫发出的第一个请求
        proxy = "http://180.175.2.68:8060"
        yield scrapy.Request(self.start_urls[0], meta={"proxy": proxy})

    def parse(self, response):
        print(response.text)
```



## 代理中间件

中间件，可以理解为spider在发送和接受请求过程中的中间环节，在这里我们可以对每个请求进行重新编辑，包括请求的header和proxy这些，都可以重写编辑再让请求发送出去，这样的好处是，我们在spider里可以专心写逻辑代码部分，请求相关的配置，可以留在中间件统一处理。

这里复习一下scrapy的架构：

![scrapy_architecture](https://xudashuai.synology.me:19000/blog/scrapy_architecture.png)



ScrapyEngine负责调度每个请求，将Spider中的每个请求链接加入到调度队列中，统一由ScrapyEngine调度，每个链接交给Downloader获取页面资源，在ScrapyEngine将链接URL交给Downloader下载之前，都要经过DownloaderMiddlewares，这里就是我们的中间件，自定义每个即将进行的请求。

### 静态IP

#### 简单介绍

静态代理中间件和上面的代码实现的功能一样，区别就是，在中间件实现的是对全局的每一个请求进行控制，上面说过，发出去的每个请求都会经过中间件，所以这里的处理是全局的（当然也可以自定义逻辑，对一些特定的请求进行处理）。

#### 示例代码

先创建一个`TestSpider`爬虫，用来测试。请求`http://httpbin.org/ip`这个网址，会返回客户端的IP地址，可以测试代理有没有生效。

```python
# -*- coding: utf-8 -*-
import scrapy


class TestSpider(scrapy.Spider):
    name = 'test'
    allowed_domains = ['test.com']
    start_urls = ['http://httpbin.org/ip']

    def parse(self, response):
        print("parse-->", response.text)
```

我们来编写一个代理中间件，中间件写在`middlewares.py`文件中，在文件最后写一个新的类，重写`process_request`函数，看名字就知道，这是用来处理请求的，函数里也很简单，给传递过来的每个`request`的`meta`都加上一个`proxy`参数。

`process_request`函数有2个参数，`request`就是spider中发送的请求，我们可以重新编辑这个请求的属性。`spider`当前运行的spider相关信息，比如爬虫的名字可以在这里获取，对于不同名称的爬虫，同一个中间件中可以做到分离处理。

```python
class TestProxyMiddleware(object):
    def process_request(self, request, spider):
        proxy = "http://47.94.89.87:3128"
        request.meta["proxy"] = proxy
        print(f"TestProxyMiddleware --> {proxy}")
```

下面来测试一下···

先将我们写的中间件打开，在`settings.py`中，打开配置：

将默认的中间件注释掉，打开我们写的，这就体现了中间件的好处，需要的时候在这里打开，不用了就注释掉。

```python
# Enable or disable downloader middlewares
# See https://docs.scrapy.org/en/latest/topics/downloader-middleware.html
DOWNLOADER_MIDDLEWARES = {
   # 'test_spider.middlewares.TestSpiderDownloaderMiddleware': 543,
   'test_spider.middlewares.TestProxyMiddleware': 543,
}
```

运行爬虫：

```shell
 ...
 'scrapy.extensions.logstats.LogStats']
2019-08-20 22:25:56 [scrapy.middleware] INFO: Enabled downloader middlewares:
['scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware',
 ...
 'test_spider.middlewares.TestProxyMiddleware',
 ...
 'scrapy.downloadermiddlewares.stats.DownloaderStats']
2019-08-20 22:25:56 [scrapy.middleware] INFO: Enabled spider middlewares:
...
2019-08-20 22:25:56 [scrapy.extensions.telnet] INFO: Telnet console listening on 127.0.0.1:6023
TestProxyMiddleware --> http://47.94.89.87:3128
2019-08-20 22:26:00 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://httpbin.org/ip> (referer: None)
parse--> {
  "origin": "47.94.89.87, 47.94.89.87"
}

2019-08-20 22:26:00 [scrapy.core.engine] INFO: Closing spider (finished)
...

```

可以看到，我们的中间件在项目中开启了，并且运行成功，代理生效。

### 动态代理

#### 简单介绍

动态代理和静态代理原理是一样的，只不过我们的需求变了，不能每个请求的IP代理是同一个，要每个都不一样，或者每段时间要变一下代理地址。其实就是每次给`request`的`meta`赋予不同的`proxy`参数，这里就需要借助`代理池`的概念，假设我们有个由IP代理组成的`代理池`，每次请求过来，我们随机从池子里取一个代理即可，只要这个池子够大，随机取代理的重复率就会很低，那些代理服务商，就是维护了一个很大的代理池供我们使用。

现在的代理商的随机IP代理都是给一个服务接口，你每次可以请求得到N个IP地址，这些地址你合理的赋给你的每个请求即可。

#### 示例代码

这里我就不写了，[贴一个帖子](https://blog.csdn.net/u010978757/article/details/83409571)



### 代理隧道

#### 简单介绍

代理隧道的工作模式：我们的请求发送到代理隧道服务器（代理商提供），隧道服务器使用随机的代理IP去获取我们需要请求的资源，再将资源发送回来。代理隧道的好处是，我们配置简单，像配置静态代理IP那样简单，却能达到随机代理IP的效果。

#### 示例代码

代码和静态代理差不多，区别在于，代理隧道一般需要进行验证，你注册某个代理服务网站，会得到一个KEY，这个KEY随着你的请求一起带给代理服务器，验证通过即可进行请求。

这里拿讯代理举例，处理代理的函数依然是`process_request`，函数里多了一个`request.headers['Proxy-Authorization'] = self.proxyAuth`参数，这就是校验用的，校验的参数是在`__init__`方法中生成的，每个服务商生成校验参数的方法不太一样，这是他们规定好的，照做就行了。

中间件启用方法和上面一样。配置成功以后，爬虫的每个请求都会转发到`https://forward.xdaili.cn:80`。

```python
class ProxyMiddleware_xun(object):
    def __init__(self):
        # 讯代理服务器
        self.proxyServer = "https://forward.xdaili.cn:80"
        # 代理隧道验证信息
        proxyUser = ""  # 订单号
        proxyPass = ""  # 秘钥

        timestamp = str(int(time.time()))  # 计算时间戳
        string = "orderno=" + proxyUser + "," + "secret=" + proxyPass + "," + "timestamp=" + timestamp
        string = string.encode()
        md5_string = hashlib.md5(string).hexdigest()  # 计算sign
        sign = md5_string.upper()  # 转换成大写

        self.proxyAuth = "sign=" + sign + "&" + "orderno=" + proxyUser + "&" + "timestamp=" + timestamp

    def process_request(self, request, spider):
        '''处理请求request'''
        request.headers['Proxy-Authorization'] = self.proxyAuth
        request.meta['proxy'] = self.proxyServer
```


### 配置多个代理随机使用

这里配置了多个代理隧道，每次请求随机使用一个，减少被ban几率。

```python
from random import choice


class SuperProxy(object):
    def proxy_kuai(self):
        # 隧道服务器
        tunnel_host = "***********"
        tunnel_port = "***********"

        # 隧道id和密码
        tid = "***********"
        password = "***********"
        proxyServer = 'http://%s:%s@%s:%s' % (tid, password, tunnel_host, tunnel_port)
        proxyAuth = "Basic %s" % (base64.b64encode(('%s:%s' % (tid, password)).encode('utf-8'))).decode('utf-8')
        return proxyServer, proxyAuth, "kuai"

    def proxy_abu(self):
        # 代理服务器
        proxyServer = "http://http-dyn.abuyun.com:9020"
        # 代理隧道验证信息
        proxyUser = "***********"
        proxyPass = "***********"
        proxyAuth = "Basic " + base64.urlsafe_b64encode(bytes((proxyUser + ":" + proxyPass), "ascii")).decode("utf8")
        return proxyServer, proxyAuth, "abu"

    def proxy_xun(self):
        # 讯代理服务器
        proxyServer = "http://forward.xdaili.cn:80"
        # 代理隧道验证信息
        proxyUser = "***********"  # 订单号
        proxyPass = "***********"  # 秘钥

        timestamp = str(int(time.time()))  # 计算时间戳
        string = "orderno=" + proxyUser + "," + "secret=" + proxyPass + "," + "timestamp=" + timestamp
        string = string.encode()
        md5_string = hashlib.md5(string).hexdigest()  # 计算sign
        sign = md5_string.upper()  # 转换成大写
        proxyAuth = "sign=" + sign + "&" + "orderno=" + proxyUser + "&" + "timestamp=" + timestamp
        return proxyServer, proxyAuth, "xun"

    def process_request(self, request, spider):
      	# 随机选择
        proxyServer, proxyAuth, name = choice([self.proxy_kuai, self.proxy_abu, self.proxy_xun])()
        request.meta["proxy"] = proxyServer
        request.headers["Proxy-Authorization"] = proxyAuth
        print("---- super请求请求 {} 正在使用代理隧道-{} 请求地址：{} ----".format(datetime.datetime.now(), name, request.url))
```


### 填坑之：代理隧道怎么都验证不成功
市面上代理隧道的验证方式多数都是在请求头中加`Proxy-Authorization`参数进行服务端的校验，如果校验不通过，则无法使用，不同的服务商对返回的错误提示定义不一样，拿我测试的`讯代理`来说，如果验证不通过，则返回`{"code":200,"msg":"auth fail, no auth header"}`，但是我们代码写的并没有问题，验证就是不通过，后来在scrapy的源码里找到了坑。
先进入scrapy的源码，如果你使用的是pycharm，查看源码步骤为：左侧的项目浏览窗口拉倒最底下，有个`External Libraries` -> `Python3.X` -> `site-packages`，里面是当前环境的所有模块，找到`scrapy`，找到文件`scrapy/core/downloader/handlers/http11.py`，打开`http11.py` Ctrl+f 搜索`def download_request`，找到下面这个函数：（中间内容我省略了）

（如果你不是使用的pycharm，自行找到你当前使用的Python的安装位置，查看`site-packages`文件夹）

```python
    def download_request(self, request):
        timeout = request.meta.get('download_timeout') or self._connectTimeout
        agent = self._get_agent(request, timeout)

        # request details
        url = urldefrag(request.url)[0]
        method = to_bytes(request.method)
        headers = TxHeaders(request.headers)
        if isinstance(agent, self._TunnelingAgent):
            headers.removeHeader(b'Proxy-Authorization')
        if request.body:
            bodyproducer = _RequestBodyProducer(request.body)
        ...
        ...
        d.addBoth(self._cb_timeout, request, url, timeout)
        return d
```

发现其中有两行代码：

```python
if isinstance(agent, self._TunnelingAgent):
            headers.removeHeader(b'Proxy-Authorization')
```

问题很显然了，我们设置的`Proxy-Authorization`参数在源码里被屏蔽掉了，导致我们的验证信息无法发送到服务端。解决办法是把这两行注释掉！