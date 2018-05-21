---
title: post请求设置cookie
date: 2018-05-21 15:17:25
tags: 
 - http 
 - cookie
categories: 技术
---
今天在做node登录模块的时候,发现昨天可以设置的cookie,今天无法设置了？QAQ
然后再看看请求里面,他的header里面是有一个set-cookie,然鹅cookie里面就是没有值- -
于是疯狂百度,终于找到了一个解决问题的回答
先说明一下 ,前端发送的是**post**请求,node 用**cors**解析

## 实际解决
这里先放一下[原文](https://segmentfault.com/q/1010000007435062?_ea=1354667)
大概的意思是,前端在发送post请求的时候,需要在请求时加上 **credentials: include**,来表示允许其它域来发送cookie(懵,get不会吗- -,看来还是太菜
嘛这里还是注意下,不同的ajax插件设置的方法也不太一样,比如我用的是axios插件那我是这么写的0.0
```js
axios.create({
        withCredentials: true // 允许携带cookie
 })
```
然后,服务端的话也要做对应的处理
`js‘Access-Control-Allow-Credentials’：true`

node这里的话由于我用的是cors模块所以就是这样去设置
```js
app.use(cors({
  credentials: true, 
}));  
```
(嘛 node不太好还是好好学习吧,另外感觉有关于这类的资源好少QAQ
