---
title: 来撸一个css3背景渐变动画
copyright: true
date: 2018-06-11 15:17:16
tags: 
- 技术
- css3
---
### css3 背景渐变动画
> 最近在做登录窗体的时候,想在登录的时候加个渐变的动画,于是就有了现在这个折腾
#### css3 不支持linear-gradient的过渡
  首先先放一下[原文](https://www.imooc.com/article/27831)
  对,木有错,并且因为这个坑我浪费了快一个小时的时间,哭T_T
  不过没关系,我们还是来先扯一下吧,要做渐变的过渡我们首先得给按钮添加一个渐变的属性
  ```css
  .button{
    width:250px;
    height:35px;
    background:linear-gradient( 135deg, #43CBFF 10%, #9708CC 100%);
  }
  ```
  嘛就是这样,兼容的话懒得添加前缀了2333,如果是ie9以下的话就只能使用filter这个属性去实现渐变的效果0.0,具体的实现各位就去百度一下吧~
  <!--more-->
#### 添加动画
  渐变的样子有了,那现在我们得让他动起来
  实际上上述的样式我们还需要做一些改变,背景用伪元素来代替,就下面这样
  ```css
      .button{
        width:250px;
        height:35px;
        position:relative;
     }
      .button::before{
         content:"";
         position: absolute;
         background:linear-gradient( 135deg, #43CBFF 10%, #9708CC 100%);
         top: -100%;
         left: -100%;
         bottom: -100%;
         right: -100%;
         background-size:100% 100%;
      }
  ```
  好,来解释下为什么,因为css3的过渡不支持linear-gradient这个的原因,于是我们用在背景添加一个渐变的伪元素。
  之后通过**移动**这个伪元素来让他有一个过渡的效果,niceヽ(✿ﾟ▽ﾟ)ノ,想法有了那我们现在就去实现他!
  ```css
      .button{
        width:250px;
        height:35px;
        position:relative;
      }
      .button::before{
         content:"";
         position: absolute;
         background:linear-gradient( 135deg, #43CBFF 10%, #9708CC 100%);
         top: -100%;
         left: -100%;
         bottom: -100%;
         right: -100%;
         background-size:100% 100%;
         animation: bgposition 6s infinite linear alternate;
      }
      @keyframes bgposition {
        0% {
            transform: translate(0%, 0%);
        }
        25% {
            transform: translate(30%, -30%);
        }
        50% {
            transform: translate(-30%, -30%);
        }
        75% {
            transform: translate(-30%, 30%);
        }
        100% {
            transform: translate(30%, 30%);
        }
      }
  ```
  ojbk,如此一来通过移动伪元素的方式实现了一个渐变的过渡~
  最近在玩electron,真是太好玩了23333
  嘛看来动画用的太少了,继续加油╰(*°▽°*)╯!

  
