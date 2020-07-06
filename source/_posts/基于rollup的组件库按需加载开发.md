---
title: 基于rollup的组件库-按需加载开发
copyright: true
date: 2020-06-28 10:34:23
tags: 
- 前端
- rollup
---
由于项目开发是自己在公司内部整的，无法开源，另外本篇仅代表个人的一些思路，有更好的做法欢迎指正，谢谢

### 选用rollup的理由
1. tree-shaking(副作用相关)
2. 多种格式输出(es, cjs)
3. 配置简单，编译可读性好(webpack会带有runtime代码，\_\_webpack_require\_\_之类的)

<!--more-->

### 按需加载
> 简单来说就是摒弃不需要的组件代码，跟路由切分的作用稍微有些不同

在组件库开发中，针对**引用了该库的项目**而言，在打包的时候就需要想办法去摒弃掉无用的代码，从而减小项目体积，那我们如何实现呢？有两种方案：  
* tree-shaking
  想办法让项目在打包的时候自己抖掉无用的组件代码，从而实现体积的优化
* 使用具体路径
  引用时写上具体的组件目录，没有引用到的组件自然不会被打包进来  

当然，这两种方式都基于我们能够给组件库做**code-splitting**, 此类特性我们也可以在[element-ui](http://element-cn.eleme.io/#/zh-CN)和[antd](https://ant.design/)中看到相关提示
#### antd
{% asset_img antd-split.jpg %}
#### element-ui
{% asset_img element-split.jpg %}

### code-splitting
  >我们也可以叫他代码切割，通过某种手段将不同的代码块区分，并打包成单独的文件   
  
  在rollup中有两种实现方式
  1. 使用在配置文件中或者npm script中使用属性**experimentalCodeSplitting**来打包(验证可行，但打包出来的文件带hash值，暂时无法使用)，并且这个属性可能是早期rollup的产物，在后面的文档中并没有找到该属性
  2. 通过解析目录，用多配置文件的方式来打包(已验证可行)，具体写法可以参考官方文档中的[Configuration Files](https://rollupjs.org/guide/en/#configuration-files)这一节，指明了配置文件可为一个数组
  {% asset_img rollup-array-output.jpg %}
  数组配置还可以使用[rollup-plugin-multi-entry](https://github.com/rollup/rollup-plugin-multi-entry)插件来完成  
#### 入口文件处理
  最后就是入口文件的处理，在编译后的组件库的入口文件处需要将所有的文件导出，效果类似下面这样
  ```js
  export { ComponentA } from './A'
  export { ComponentB } from './B'
  ```
  不过在自己实现的时候，rollup会根据打包时的入口文件静态依赖关系，将引用到的组件都打包到一个组件中
  如果是这样的话，入口文件就不能够达到我们想要的**tree-shaking**的效果 
  {% asset_img images.jpg %} 
  那就另辟蹊径一下吧，这个入口文件我们能不能自己生成呢？
  我们需要知道所要导出的所有组件，在文件结构规范的情况下，需要导出的目录可以通过扫描 **src**目录实现  
  我们要做的只是如下一样将他写入到build目录就可以了

  {% asset_img code-example.jpg %}

  然后我就顺手写了个[小插件](https://github.com/OctupleSakura/rollup-generate-entry)，尽管我不并不知道这思路合不合理hhh
  不过至少，它已经勉强能用了:laughing:

### 参考资料
1. [rollup](https://rollupjs.org/guide/en/)
2. [antd](https://ant.design/docs/react/introduce-cn)
3. [babel-plugin-import](https://github.com/ant-design/babel-plugin-import)
4. [babel-plugin-component](https://github.com/ElementUI/babel-plugin-component)