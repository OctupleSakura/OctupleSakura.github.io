---
title: 来扒一扒JS的继承
copyright: true
date: 2018-06-05 09:30:52
tags:
- 技术
- JS
categories: 技术
---
> 为什么要写这一篇呢?原因是之前面试时卡在了一道继承的题上面,并且挺多公司会涉及到面向对象的概念,于是决定还是要把继承再过一遍
###  JS继承
  首先我们先来说一下概念,来从理论上了解继承是个什么东西0.0  
  继承是基于js面向对象为基础的情况下的一个机制,将某个定义好的类作为基类,创建的子类将继承父类的所有方法和属性,此外子类可以添加自身的属性和添加方法,因此可以把父类当做一个模板类。  
  那么继承到底在什么样的应用场景下适合使用呢QAQ？  
  嘛 毕竟对于大多数人来说没有做过的东西只有概念的话肯定是不知道怎么去用的。  
  网上搜了很多也没有这方面的应用,其实我自己用的也不太好，但我还是举个栗子吧~
  <!--more-->
  **应用场景**
  首先比方说我们需要做一个游戏,游戏的话肯定需要很多的角色,那么我们首先就需要做一个所有角色的基类,所有角色的特性都是基于这个角色类来进行特性赋予。
  那我直接贴下代码来看看是个什么样子(๑•̀ㅂ•́)و✧
  ```js
    function Person(name){
       this.name = name||'爸爸';
    }
    Person.prototype.walk = function(direction){
       console.log(this.name + ' is walking on the ' + direction);      
    }
    const PersonA = new Person('爸爸');
    PersonA.walk('left')
    //>> 爸爸 is walking on the left
  ```
  好,这样子就定义出了一个会走路的爸爸了0 0  
  接下来,我们每个角色都需要一些不一样的超能力,比如第一个角色会tm喷火！那我们就需要在普通角色的基础上给他添加上喷火的功能,比如下面这样
  ```js
    function FirePerson(data){
      Person.call(this);
      this.name = data.name;
      this.height = data.height;
    }
    FirePerson.prototype = new Person();
    FirePerson.prototype.constructor = FirePerson;
    FirePerson.prototype.fire = function(){
      console.log('我tm喷火!');
    }
    const FirePersonA = new FirePerson({name:'F',height:'180'});
    FirePersonA.walk('right');
    FirePersonA.fire();
    //>>F is walking on the right
    //>>我tm喷火!
  ```
  好,这样一来这个角色不仅会走路,还会喷火了。这样一来我们在初始角色的基础上可以添加很多很多的能力ヽ(✿ﾟ▽ﾟ)ノ  
  嘛当然这样还是不够的,因为ES6也是有继承的,并且ES6的class仅仅只是ES5原型的一个语法糖,本质上并没有区别(文档上好像是这么写的0 0)  
  那我们ES6的方式来实现一遍  
  ```js
  class Person {
    //构造函数
    constructor(name) {
      this.name = name || '爸爸';
    }

    walk(direction){
       console.log(this.name + ' is walking on the ' + direction);      
    }
  }
  const PersonA = new Person();

  class FirePerson extends Person{
    constructor(name){
      super();
      this.name = name || 'F';
    }
    fire(direction){
       console.log('我tm喷火!');     
    }
  }
  const FirePersonA = new FirePerson('F');
  ```
  好,然后实现了同样的功能,这样就大概可以了解继承的一个思想了吧(其实这是写给我自己看的233)  

  嘻嘻,最近从新概念1开始恶补英语,这一年要加油咯！

  
  


  
