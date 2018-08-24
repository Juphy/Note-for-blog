---
title: nodejs发起http请求
date: 2018-08-23 11:22:00
categories: #文章分类
- node
tags:  
- fetch
- http
- request
- axios
toc: true # 生成目录
---

【原文】[5 Ways to Make HTTP Requests in Node.js](https://www.twilio.com/blog/2017/08/http-requests-in-node-js.html)
## HTTP模块，http或者https库 需要解析JSON格式
```
const https = require('https');
https.get('https://api.nasa.gov/planetary/apod?api_key=DEMO_KEY', res => {
    let data = '';
    res.on('data', chunk => {
        data += chunk;
    })
    res.on('end', () => {
        console.log(1, JSON.parse(data))
    })
}).on('error', err => {
    console.log('Error：' + err.message);
})
```

## request库，如果使用promise可以调用request-promise库
```
const request = require('request');
request('https://api.nasa.gov/planetary/apod?api_key=DEMO_KEY', {json: true}, (err, res, body) => {
    if (err) {
        return console.log(err)
    }
    console.log(2, body);
})
```

## request-promise库 request的promise操作
```
const requestPromise = require('request-promise');
requestPromise('https://api.nasa.gov/planetary/apod?api_key=DEMO_KEY')
    .then(res => {
        console.log(3, res);
    }).catch(err => {
    console.log(err);
});
```
## axios
Axios是一个基于promise的HTTP客户端，可以用于浏览器和Node.js

### axios库 默认情况下，Axios可以解析JSON响应
```
const axios = require('axios');
axios.get('https://api.nasa.gov/planetary/apod?api_key=DEMO_KEY')
    .then(res => {
        console.log(4, res.data);
    }).catch(err => {
    console.log(err);
})
```

### axios库的all，发起并发请求
```
axios.all([
    axios.get('https://api.nasa.gov/planetary/apod?api_key=DEMO_KEY&date=2017-08-03'),
    axios.get('https://api.nasa.gov/planetary/apod?api_key=DEMO_KEY&date=2017-08-02'),
]).then(axios.spread((res1, res2) => {
    console.log('res1', res1.data);
    console.log('res2', res2.data);
})).catch(err => {
    console.log(err);
})
```

## superAgent 默认解析JSON响应，能进行链式调用
```
const superagent = require('superagent');
superagent.get('https://api.nasa.gov/planetary/apod')
    .query({api_key: 'DEMO_KEY', date: '2017-08-02'})
    .end((err, res) => {
        if (err) {
            return console.log(err)
        }
        console.log(5, res.body);
    })
```

## Got 类似于promise，比较轻量，不像request那样臃肿
```
const got = require('got');
got('https://api.nasa.gov/planetary/apod?api_key=DEMO_KEY', {json: true})
    .then(res => {
        console.log(res.body);
    }).catch(err => {
    console.log(err.reponse.body);
});
```

## node-fetch库与window.fetch API保持一致
```
const fetch = require('node-fetch');
// plain text or html
fetch('https://github.com/')
    .then(res => res.text())
    .then(body => console.log(body));
// json
fetch('https://api.github.com/users/github')
    .then(res => res.json())
    .then(json => console.log(json));
// catch network error
fetch('http://domain.invalid/')
    .catch(err => console.error(err));
// stream
fetch('https://assets-cdn.github.com/images/modules/logos_page/Octocat.png')
    .then(res => {
        return new Promise((resolve, reject) => {
            const dest = fs.createWriteStream('./octocat.png');
            res.body.pipe(dest);
            res.body.on('error', err => {
                reject(err);
            });
            dest.on('finish', () => {
                resolve();
            });
            dest.on('error', err => {
                reject(err);
            });
        });
    });
```    
[更多fetch的用法](https://www.npmjs.com/package/node-fetch)