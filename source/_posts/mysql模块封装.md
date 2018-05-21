---
title: mysql模块封装
date: 2018-05-14 17:26:34
tags: 
- node.js
- node
- mysql
categories: 技术
---

## 前言
这两天都在捣鼓node,给项目做一个用户登录,用户注册加密的部分,涉及到sql这个地方也踩了大大小小的坑0.0
最开始是打算用node的一个orm框架--[sequelize](http://docs.sequelizejs.com/)去做mysql的处理,然鹅还是放弃了(其实是卡在了一个不懂的错误上,也没有仔细去研究),有兴趣的朋友可以去玩一下QAQ
总之最后,还是自己对mysql模块封装,完成了一个用户注册的功能

## 具体封装实现
我这里直接贴下代码
```js
//数据库函数,返回一个promise对象
const mysqlConnect = (sql,params) =>{
  const connection = mysql.createConnection({
       host:"host",
       port:3306,
       user:"user",
       password:"password",
       database:"database"
    });
  connection.connect();
  //返回两个promise 对象 用来拓展mysql函数操作
  return {
      query:(sql,params)=>{
        return new Promise((resolve, reject)=>{
          connection.query(sql,params,(err,rows,fields)=>{
            if(err){
                console.log(err);
                return;
            }
            resolve(rows)
        })
       })
      },
      end:()=>{
        return new Promise((resolve,reject)=>{
          connection.end((e)=>{
            if(e){
              console.log("mysql end error!");
              return;
            }
            console.log("mysql end success!");
            resolve(1);
          });
        })
      }
  }
}
```
执行mysqlConnect函数,会连接数据库,并且会返回两个函数.一个是query,用来执行sql语句,一个是end用来结束连接。

## 封装调用
这里放一下调用,作为参考0 0
```js
 const insertSql = 'INSERT INTO music_user VALUES (?,?,"111",?,?,NOW(),"0")';//插入用户数据的sql
 let params = [UserId[0].uid,ctx.query.username,password[1],password[0]];//传入用户数据参数
 const insert = await User.query(insertSql,params);//执行存储过程
 await User.end();//结束连接
```
这里的UserId[0]是代码前面的部分有生成的uuid,passord是node加密处理过的密码和盐值
大概就是上面这样子,之前mysql模块的回调实在头疼,趁着这个项目有重复的需求赶紧把自己之前的代码又重构了一遍,嘛对于node这块也是一个新手,如果有大佬看到的话不足之处也烦请多多指教啦~
