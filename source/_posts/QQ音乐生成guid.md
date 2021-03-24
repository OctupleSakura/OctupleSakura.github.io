---
title: QQ音乐生成guid
date: 2018-05-14 17:40:08
tags: 
- 前端
- QQ音乐
categories: 技术
---
因为之前在用的一个接口突然就403了,因此不得不解析一下QQ音乐原先是怎么获取到音乐资源,QQ音乐的搜索等等应该都挺好拿到的,因为那边没有做太多处理所以就不写了
<!-- more -->
## 生成guid
首先先放一下翻了好多文章才找到的一个方法

```js
getGuid(){
   var t = (new Date).getUTCMilliseconds();
   if(document.cookie.indexOf("pgv_pvid")!=-1){
     return this.getCookie("pgv_pvid");
   }
   let  _guid = Math.round(2147483647 * Math.random()) * t % 1e10;
   document.cookie ="pgv_pvid=" + _guid + "; Expires=Sun, 18 Jan 2038 00:00:00 GMT;PATH=/;";
   return _guid
}

```

这个是在原文的方法上根据自己的需要进行的一个处理,生成guid之后需要将他放入cookie中并且将这个guid用于请求vkey上
这里还做了一个判断,如果没有cookie的话就进行再生成.否则就一直用cookie里面的guid

## 获取vkey
这里首先放一个链接,用来请求[资源](https://c.y.qq.com/base/fcgi-bin/fcg_music_express_mobile3.fcg?g_tk=1278911659&hostUin=0&format=jsonp&callback=callback&inCharset=utf8&outCharset=utf-8&notice=0&platform=yqq&needNewCode=0&cid=205361747&uin=0&songmid='+songmid+'&filename=C400'+songmid+'.m4a&guid='+guid)

这里的话需要设置的变量有两个,一个是生成的guid,另外一个是songmid,这里是获取到音乐数据的list时候里面会有这个id.同样放进去就好了,之后会返回一个数据,这里也放一下大概的格式
```js
{config:{},data:"{}",header:{},request:{},status:200,statusText:"OK"}
```
其中data里面就包含了根据guid请求到的vkey,这样就拿到了我们要的key了

## 根据vkey和guid来请求音乐资源
拿到了vkey和guid,这里就可以直接给audio设置src了,这里放下[路径](http://dl.stream.qqmusic.qq.com/C400002WLzlO1fjnXZ.m4a?guid=6238123201&vkey=45906EDC7FB59B1A8E4420E2403106133F9669AC2F6F71AD43ECA6C85043EA652F14CA06C4F8E8B48BA17D6C447F19FAD8D5B1C36297457E&uin=0&fromtag=38)
这个地方要设置的只有vkey和guid 就是上面两个步骤拿到的
要注意这里的vkey一定是要通过guid生成请求出来的才可以拿到,如果是另一个guid请求到的vkey,那么可能拿不到音乐资源(大概吧hhhh我也没有试过

## 参考资料
[guid方法的原文](http://www.bubuko.com/infodetail-2309843.html)

