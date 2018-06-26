---
title: node后端转发请求踩坑
copyright: true
date: 2018-06-26 11:04:49
tags:
- node
- 技术
categories: 技术
---
### node后端转发请求踩坑
最近想起来了要将之前的一个个人作品用gp挂起来,但是请求资源的地方用的本地代理进行跨域,于是又开始折腾node这里来进行一个后端的代理跨域获取资源T_T  
### http.request
这个是后端http模块的一个方法,实际上爬虫也是用这个来获取到页面的资源然后进行解析,不过这里的话我们只是先用他来请求资源,先上代码0 0  
<!--more-->
```js
const http = require('http');
const koa = require('koa');
const route = require('koa-route');
const cors = require('koa-cors'); 
const app = new koa();

//解决跨域问题
app.use(cors({
  credentials: true, 
}));  

const index = async ctx => {
  let options = { 
     hostname: 'c.y.qq.com', 
     port: 80, 
     path: '/musichall/fcgi-bin/fcg_yqqhomepagerecommend.fcg?g_tk=5381&uin=0&format=json&inCharset=utf-8&outCharset=utf-8&notice=0&platform=h5&needNewCode=1&_=1521092161716"`', 
     method: 'GET' 
  }; 
  // write data to request body  
  let data = await request(options,'index');
  ctx.response.body = data;
}

function request(options,type){
  return new Promise((resolve,reject)=>{
    let ser = http.request(options,function (res) {
      let str = '';
      console.log('STATUS: ' + res.statusCode);  
      console.log('HEADERS: ' + JSON.stringify(res.headers));  
      res.setEncoding('utf8');  
      res.on('data', function (chunk) {  
        //解决json数据获取不完整的问题
        str += chunk;
      }); 
      res.on('end',function(){
          resolve(str);
      }) 
    })
    ser.on('error', function (e) {  
      reject(e.message);  
    });
    ser.end();
  })

app.use(route.get('/proxy/index', index));

app.listen(1341);
```

ok这里是部分源码,然后这里还有几个坑,我来说明一下0.0  
首先实际上要请求的资源不止这一个,所以这里用koa来做一个路由方便前台请求。  
1. ctx对象无法获取  
在做这部分的时候在request里面直接ctx.response.body = xxx,前台会报错(错误忘记截图了哭),调试的时候发现似乎是因为request里面无法获取到ctx对象,于是就把http.request用Promise包装了一遍,等待请求资源完成之后再将数据返回,同时这样也会比较清晰0.0  
2. 返回json不完整
在写另外一个接口的时候,出现了返回json不完整的问题,哭死,但有问题还是要解决的,于是找到了一个方法,这里是[原文](https://cnodejs.org/topic/5665541e63dc988306f2917d),于是就有了代码中str的那个部分,就是为了解决**返回json不完整**的问题。  
  
实际上还踩了各种各样的坑2333,不过不主攻后端于是在找到解决方法之后便没有深究,记录下来,今后继续努力吧~
