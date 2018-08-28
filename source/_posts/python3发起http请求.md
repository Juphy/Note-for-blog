---
title: python3发起http请求
date: 2018-08-24 11:22:00
categories: #文章分类
- python
tags:  
- urllib
- requests
toc: true # 生成目录
---

## urllib
> python内置的标准库模块

### urlopen
```
import urllib.request
import urllib.parse

url = 'https://api.github.com/users/github'
f = urllib.request.urlopen(url)
// html或者json数据都可以解析
print(f.read().decode('utf-8'))
```
利用type(f)可以查看输出响应的类型，`<class 'http.client.HTTPResponse'>`，它是一个HTTPResponse类型的对象，它主要包括read()、readinto()、getheader(name)、getheaders()、fileno()等方法，以及msg、version、status、reason、debuglevel、closed等属性。

- 调用read()方法可以得到返回的网页内容
- 调用status属性可以得到返回结果的状态码，如200代表请求成功，404代表网页未找到等
- getheaders()响应头信息

`urlopen()`也可以传递参数：
```
urllib.request.urlopen(url, data=None, [timeout, ]*, cafile=None, capath=None, cadefault=False, context=None)
```
- data
data参数是可选的。如果要添加该参数，并且如果它是字节流编码格式的内容，即bytes类型，则需要通过bytes()方法转化。另外，如果传递了这个参数，则它的请求方式就不再是GET方式，而是POST方式。

```
import urllib.parse
import urllib.request
 
data = bytes(urllib.parse.urlencode({'word': 'hello'}), encoding='utf8')
response = urllib.request.urlopen('http://httpbin.org/post', data=data)
print(response.read().decode('utf-8')) // 如果不加decode('utf-8')

打印结果：
{
  "args": {},
  "data": "",
  "files": {},
  "form": {
    "word": "hello"
  },
  "headers": {
    "Accept-Encoding": "identity",
    "Connection": "close",
    "Content-Length": "10",
    "Content-Type": "application/x-www-form-urlencoded",
    "Host": "httpbin.org",
    "User-Agent": "Python-urllib/3.6"
  },
  "json": null,
  "origin": "61.148.29.62",
  "url": "http://httpbin.org/post"
}
```
传递的参数{word: 'hello'}，它需要被转码成bytes（字节流）类型。其中转字节流采用了bytes()方法，该方法的第一个参数需要是str（字符串）类型，需要用urllib.parse模块里的urlencode()方法来将参数字典转化为字符串；第二个参数指定编码格式，这里指定为utf8。

- timeout
用于设置超时时间，单位为秒，意思就是如果请求超出了设置的这个时间，还没有得到响应，就会抛出异常。如果不指定该参数，就会使用全局默认时间。它支持HTTP、HTTPS、FTP请求。

```
import urllib.request
 
response = urllib.request.urlopen('http://httpbin.org/get', timeout=0.1)
print(response.read())

运行结果：
超时报错，服务器没有响应，于是抛出了URLError异常。该异常属于urllib.error模块，错误原因是超时。
```
可以通过设置这个超时时间来控制一个网页如果长时间未响应，就跳过它的抓取。这可以利用try except语句来实现，相关代码如下：
```
import socket
import urllib.request
import urllib.error
 
try:
    response = urllib.request.urlopen('http://httpbin.org/get', timeout=0.1)
except urllib.error.URLError as e:
    if isinstance(e.reason, socket.timeout):
        print('TIME OUT')
```
[更多urlopen的用法](https://docs.python.org/3/library/urllib.request.html)

### request

简单的请求可以使用urlopen，如果需要在请求中加入Headers等信息，就可以使用更强大的Request类来构建。
```
import urllib.request
 
request = urllib.request.Request('https://python.org')
response = urllib.request.urlopen(request)
print(response.read().decode('utf-8'))
```
依然是使用urlopen发送请求，只不过参数不再是url，而是一个request类型的对象。

> class urllib.request.Request(url, data=None, headers={}, origin_req_host=None, unverifiable=False, method=None)

- url 这是必选，其他事可选
- data 必须是bytes（字节流）类型，如果是字典，可以先用urllib.parse模块里的urlencode()编码。
- headers 可以在构造请求时通过headers参数直接构造，也可以通过调用请求实例的add_header()方法添加。
- origin_req_host 指的是请求方的host名称或者IP地址。
- unverifiable表示这个请求是否是无法验证的，默认是False，意思就是说用户没有足够权限来选择接收这个请求的结果。
- method是一个字符串，用来指示请求使用的方法，比如GET、POST和PUT等
```
from urllib import request, parse
 
url = 'http://httpbin.org/post'
headers = {
    'User-Agent': 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)',
    'Host': 'httpbin.org'
}
dict = {
    'name': 'Germey'
}
data = bytes(parse.urlencode(dict), encoding='utf8')
req = request.Request(url=url, data=data, headers=headers, method='POST')
response = request.urlopen(req)
print(response.read().decode('utf-8'))

url即请求URL，headers中指定了User-Agent和Host，参数data用urlencode()和bytes()方法转成字节流。另外，指定了请求方式为POST。
```

或者使用add_header()
```
req = request.Request(url=url, data=data, method='POST')
req.add_header('User-Agent', 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)')
```

## Requests 
> 由urllib3提供支持，Requests抽象了大量的程式化的代码，使得http请求比使用内置urllib库更简单
```
// 安装requests，pip install requests

import requests
from pprint import pprint

r = requests.get(url)
pprint(r.json())
```
