---
title: 让我们用rollup来摸鱼吧
copyright: true
date: 2019-06-26 16:31:25
tags:
categories: 技术
---
**rollup**是个什么东东，和**webpack**一样,是我js模块打包器哒！但是更加的轻量级，并且上手十分简单

## 和webpack的对比
### 1. tree-shaking
利用es模块的静态分析，以及作用域分析等等，找到无用的代码，在打包的时候删除，从而达到缩减代码体积的目的，这也是为什么如果采用了**commonjs**的模式便无法进行**tree-shaking**的原因  
而这一方面rollup首先提出并实现，webpack从2开始支持，并且ugilfy也支持做这个工作，babel更新babel7，之后也优化了关于一些副作用上面的问题，同时rollup做了程序流分析的原因，这一方面做得比较好  
<!--more-->
实际上目前的打包工具的tree-shaking都会有些缺陷，原因基本都是因为函数的**副作用**导致，并且经过babel打包过后，如果你定义了类的话，为了符合es6的语义在编译之后会有一个 **_createClass**函数，函数里面采用**Object.defineProperty**来定义类里面的方法，从而产生了副作用，无法被**tree-shaking**
<img src="https://github.com/OctupleSakura/showImg/blob/master/blog/rollup/parse.png?raw=true" style="border: 1px solid #eeeeee;margin: 10px 0;"/>
而如果开启babel的**loose模式**的话，会直接通过在原型链上定义方法的形式来实现，因此loose模式下的类定义基本可以被正确的**tree-shaking**   
<img src="https://github.com/OctupleSakura/showImg/blob/master/blog/rollup/loose.png?raw=true" style="border: 1px solid #eeeeee;margin: 10px 0;"/>
有兴趣的话可以用babel的线上编译来做尝试，大体就不深究了，可以展开很长，这篇文章目的也不是主要探讨这个

#### 2. 配置
  从webpack和rollup的文档对比来看，显而易见，webpack要比rollup繁杂的多，同时rollup也没有loader的这样的说法,基本所有的编译工作都是依赖于plugin来完成，而webpack的编译工作则基本依赖于loader，plugins来做拓展和优化等loader无法做到的事情。  
  <img src="https://github.com/OctupleSakura/showImg/blob/master/blog/rollup/plugins.png?raw=true" style="border: 1px solid #eeeeee;margin: 10px 0;"/>
  并且webpack在一些功能的配置项上要比rollup要多一些，比如**externals**，rollup只能选择dep或者peer全部做externals，而webpack可以选择单独对某个模块执行
     
    
## rollup适合做什么
  为什么要说rollup适合做类库，而webpack适合做应用呢，有不少的原因，rollup相对于webpack一些核心功能的缺失，比如
  **代码切割**和**hmr**，以及**静态资源的处理**，这些原因决定了rollup并不适合一个大型的应用。   

  > ps: 写博客的时候发现rollup已经有了针对es模块做代码切割的实验性语法，通过在npm script加上--experimentalCodeSplitting来实现，另外常规的代码切割需要通过import()来实现，给中文文档坑了...  

  同时，rollup在模块的一些处理上还是有一些坑的，rollup默认支持es模块的处理，对于commonjs的模块需要额外的插件，另
  外在代码压缩的时候，一般都会想到**uglifyjs**，实际上他也只支持**es5**，uglify-es虽然支持es6，但是也已经停止维护。
    <img src="https://github.com/OctupleSakura/showImg/blob/master/blog/rollup/uglify.png?raw=true" style="border: 1px solid #eeeeee;margin: 10px 0;"/>
  所以使用rollup做压缩代码的时候，更好的选择是**terser**，他是**uglify-es**的一个分支，支持es6+，并且如今也还在继续更新。
  另外如果要做一些ui库的话还是更推荐webpack，像按需加载这种功能基本上是不能在rollup上实现的，调用了其中的一个组
  件，项目打包的时候会把所有的内容都给打包进去，这种场景还是webpack更适合一些。
  
## rollup的一些优化
  在优化rollup的代码上我们需要做一些什么样的工作？
### 1. external
rollup的external需要通过插件来实现，公司项目写库的时候用的是**rollup-plugin-peer-deps-external**这个组件来
配置，基本上也很简单，最多只用配置下面两个选项    
  
- includeDependencies(是否包含dep)  
- packageJsonPath(packjson路径)  

它主要针对的是**peerDependencies**这个东西，一般是写插件或者库才会用到的一个package的属性
    <img src="https://github.com/OctupleSakura/showImg/blob/master/blog/rollup/peer.png?raw=true" style="border: 1px solid #eeeeee;margin: 10px 0;"/>
  值得一提的是这个属性在npm3中已经被取消了，但是yarn中还在继续使用，它的作用是为了让声明项目的核心依赖库，当有人在
  调用你的项目时，如果你已经安装过peerDependencies所包  含的依赖库的话则忽略，如果没有则安装，并且可以避免用户多
  次下载同一插件(这个地方好像npm已经做过优化了= =)
  除了这个以外还有**rollup-plugin-auto-external**这个插件，但是我木有用过，看了配置项应该也差不多
### 2. uglify和terser
  上面已经大概说过uglify和terser的一些关系，就不再多赘述了，基本上插件里面添加一遍就完事了，具体的配置项就直接看源
  项目的[说明](https://github.com/TrySound/rollup-plugin-terser#readme)吧   
 
  其余就是**sourcemap**之类的一些优化了，判断一下是否是生产环境就o啦~

## 参考链接
 关于[tree-shaking](https://juejin.im/post/5a5652d8f265da3e497ff3de)
 关于[peerDependencies](https://github.com/SamHwang1990/blog/issues/7)