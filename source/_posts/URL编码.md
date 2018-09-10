---
title: URL编码
categories:
  - JavaScript
tags:
  - URL
toc: true
abbrlink: 942c420c
date: 2018-09-10 14:56:00
---
## URL
一般来说，URL只能使用英文字母、阿拉伯数字和某些标点符号，不能使用其他文字和符号。

> 只有字母和数字[0-9a-zA-Z]、一些特殊符号"$-_.+!*'(),"[不包括双引号]、以及某些保留字，才可以不经过编码直接用于URL。

## URL编码
### 网址路径
`http://zh.wikipedia.org/wiki/春节`的实际地址是`https://zh.wikipedia.org/wiki/%E6%98%A5%E8%8A%82`。

"春"和"节"的utf-8编码分别是"E6 98 A5"和"E8 8A 82"，因此，"%E6%98%A5%E8%8A%82"就是按照顺序，在每个字节前加上%而得到的。

> 网址路径的编码，用的是utf-8编码

### 查询字符串

`http://www.baidu.com/s?wd=春节`，各浏览器编码不一致，用的是操作系统的默认编码。

### GET方法生成的URL
在已打开的网页上，直接用Get或Post方法发出HTTP请求。

> GET和POST方法的编码，用的是网页的编码

### Ajax调用的URL

在Ajax调用中，IE总是采用GB2312编码（操作系统的默认编码），而Firefox总是采用utf-8编码。

## escape()
除了ASCII字母、数字、标点符号"@ * _ + - . /"以外，对其他所有字符进行编码。在\u0000到\u00ff之间的符号被转成%xx的形式，其余符号被转成%uxxxx的形式。对应的解码函数是unescape()。

escape()不对"+"编码。

## encodeURI()
它着眼于对整个URL进行编码，因此除了常见的符号以外，对其他一些在网址中有特殊含义的符号"; / ? : @ & = + $ , #"，也不进行编码。编码后，它输出符号的utf-8形式，并且在每个字节前加上%。

解码函数是decodeURI()。

不对单引号'编码。

## encodeURIComponent()
用于对URL的组成部分进行个别编码，而不用于对整个URL进行编码。因此，"; / ? : @ & = + $ , #"，这些在encodeURI()中不被编码的符号，在encodeURIComponent()中统统会被编码。至于具体的编码方法，两者是一样。

解码函数是decodeURIComponent()。
