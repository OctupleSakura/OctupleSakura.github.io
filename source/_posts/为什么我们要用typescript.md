---
title: 为什么我们要用typescript
copyright: true
date: 2019-10-09 11:58:59
tags:
- 前端
- typescript
categories: 技术
---

如果让我选一个用了就离不开的东西，那么选项当中一定会有一个东西。
那就是**typescript**

<!--more-->

### 前言
ts对于前端开发来说是个门槛。为什么呢？它的学习成本并不低，并且现状是，大部分人一开始都会对着满屏幕的红色波浪线一脸懵逼，不知所措，然后放弃。  

实际上，半年前的我也是抱着这样的心态，因此，总得有足够充分的理由来让我们在学习成本比较高的情况下仍然去学习它，毕竟js确实已经够用了。  
<img 
  src="https://github.com/OctupleSakura/show-img/raw/master/blog/ts/faker-ll.jpg" style="width:300px;margin: 0;"
/>

超级感谢leader在这方面给我的帮助(已经是前leader了T_T)，真的学习到了很多。
在这里，我强烈安利这个骚东西，它确确实实的能够提升你的开发体验，且听我慢慢道来。

### ts好玩在哪里
先根据自己的经验来扯一下ts到底有什么吸引了我。

#### 规避类型引起的报错
没有对比就没有伤害，诚然，js较为宽泛的语法降低了入门的门槛，但同时也有一定的弊端，相信用js的老哥们偶尔会遇到类型问题，引发报错，不胜其烦。
ts在这方面做了很好的补足，通过给变量规定类型，很好的规避了很多许多类型相关的低级错误。
<img 
  src="https://github.com/OctupleSakura/show-img/raw/master/blog/ts/var-type.jpg" style="width:400px;margin: 0;margin-top: 10px;" 
/>

另外一些静态类型该有的，ts也能给你提供，比如常见的interface，enum，泛型，装饰器等等，在此之上，还包含很多通过ts基础语法实现的一套api，例如Pick，Extract，Extract巴拉巴拉。
因此在组件传值，接口数据方面很多都可以应用到，十分便利。

#### 代码即文档
ts有一部分的好处还体现在**接口数据**上，在写接口的过程中，如果没有文档或者注释的话，很难知道一个接口到底会有什么数据。  
如果通过ts来给接口返回数据编写类型，只需要看编辑器的提示就能知道一个接口到底需要什么参数，会返回什么数据。
不仅如此，在遇到一些复杂接口的时候，往往需要一定的**理解成本**，通过ts对接口做一些简单的设计，能够使理解成本降低，对我们开发体验也有着很大的帮助。  
> 推荐一个[小工具](http://www.json2ts.com/)，可以把json转换成ts的类型。  

举个简单的栗子0.0
如果我们在做一个发送消息的接口，这个消息有不同的类型，例如文字消息、音频消息、视频消息等等，我们要去怎么设计它？
<img 
  src="https://github.com/OctupleSakura/show-img/raw/master/blog/ts/jojo.jpg" style="width:200px;margin: 0;margin-top: 10px;" 
/>
相信你肯定就一下想到了做一个message的base interface，然后通过extends来实现各种不同类型的消息。
nice，那先贴一下我的想法，有更好的思路还请多多指教~
```js
/*
  ** 类型文件
*/

// 通过keyof关键字来拿到form类型的key值
export type TMessageKey = keyof IMessageType

// 将可能用到的类型放到一个interface里面
export interface IMessageType {
  voice: IAudioSendMessage
  text: ITextSendMessage
  image: IImageMessage
}

/*
  ** 接口层
*/

// 通过泛型限定在key当中，然后同时可以根据泛型来拿到对应的interface
public static MessageSend<T extends TMessageKey>( 
  form: IMessageType[T]
) {
  //...发送消息
}

/*
  ** 逻辑层
*/

// 通过手动的传入一个类型，明确的声明所需要传的值
MessageSend<'image'>({
  // image form data
})
```
通过类型的推演，使得我们在使用**MessageSend**接口时，只需要修改传递的消息类型，就可以得到不一样的参数类型。
真香！

#### 合理的设计
在工作当中曾经遇到这样一个问题，我想设计一个**单选框组件**。
对应的，我需要一个**default**值(默认选中)，以及**item**(每个选项包含的一些参数)，那么问题来了，如何通过ts去限制default值在item的范围内呢？

```js
interface option<S> {
  value: S
  label: string
  key: string | number
}

interface IRadioProps<V, T> {
  item: V[]
  defaultValue?: T
}

export function Radio<
  T extends V['value'],
  V extends item<S>,
  S extends string
>(props: IRadioProps<V, T>) {
  //...
}
```
> 感谢学长，这个问题之前咨询过他，还特意帮我到stackoverflow提问了，具体的回答可以看[这里](https://stackoverflow.com/questions/56651130/is-there-possible-to-determine-parameter-is-one-of-the-given-collection-types)  

答案也很简单，通过**泛型**，泛型是什么就不赘述了，直接去看ts的文档便能了解。  
通过将泛型限制在item的value中来使得defaultValue一定为item中的某个value，并给出对应的提示，让组件限制更加合理。

### 结尾
放下typescript的[官网](https://www.typescriptlang.org/docs/home.html)。
总之，ts是一个非常值得学习的东西，不仅能提升开发体验，同时能够让前端入门的小伙伴了解到类型系统，一举两得。
