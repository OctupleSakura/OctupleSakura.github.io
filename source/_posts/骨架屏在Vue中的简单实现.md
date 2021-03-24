---
title: 骨架屏在Vue中的简单实现
copyright: true
date: 2019-01-25 10:24:12
tags: 
- 前端
- vue 
categories: 技术
---
为什么需要骨架屏？骨架屏其实是一项非常不错的优化，可以有效减少pwa项目在渲染dom过程中的白屏时间，相对于loading来说好处在于能让用户感受到项目是在逐步加载，对于用户的感知会显得更加流畅  
<!--more-->
![facebook](https://github.com/OctupleSakura/show-img/raw/master/blog/skeleton/skeleton.jpg)
我们在打开facebook的时候也可以看到这玩意儿，尽管他们很早就用上了  
### 实现
首先先来说明一下我们使用的插件[vue-skeleton-webpack-plugin](https://github.com/lavas-project/vue-skeleton-webpack-plugin)，是lavas的webpack相关插件  
大体分为**3步**:  
 1. 创建骨架屏vue文件
 2. 创建skeleton的实例
 3. 在webpack的配置文件中导入skeleton实例

首先我们需要写一个能让webpack去渲染的一个骨架屏模板，就是skeleton.vue,样式可以自己想，这里就直接拿官方的demo来用了
```js
//skeleton.vue
<template>
  <div class="skeleton-wrapper">
    <section class="skeleton-block">
      <!-- eslint-disable vue/max-len -->
    </section>
  </div>
</template>

<script>
export default {
  name: 'Skeleton',
};
</script>

<style scoped>
.skeleton-block {
  display: flex;
  flex-direction: column;
  padding: 16px;
}
</style>
```

创建skeleton实例文件，导入刚刚的文件
```js
import Vue from 'vue';
import Skeleton from './Skeleton.vue';

export default new Vue({
  components: {
    Skeleton,
  },
  render: h => h(Skeleton),
});
```

最后，在vue.config.js里面，用插件去引用实例
```js
//这里要先把插件安装好
const SkeletonWebpackPlugin = require('vue-skeleton-webpack-plugin')
let option = {
  configureWebpack: {
    plugins: [
      //对应插件
      new SkeletonWebpackPlugin({
        webpackConfig: {
          entry: {
            //入口文件路径
            app: path.join(__dirname, './src/skeleton.entry.js'),
          },
        },
        //quiet 选填，在服务端渲染时是否需要输出信息到控制台
        quiet: true,
        //minimize 选填 SPA 下是否需要压缩注入 HTML 的 JS 代码
        minmize: true
      })
    ]
  },
}
```

反正直接照着官方库中的[examples](https://github.com/lavas-project/vue-skeleton-webpack-plugin/tree/master/examples)写就完了，很详细了(逃

### 踩过的坑
这里要说明一下，在vue-cli下，实际上[vue-skeleton-webpack-plugin](https://github.com/lavas-project/vue-skeleton-webpack-plugin)这个插件在非线上环境下是有坑的,skeleton的样式会渲染不出来，在官方库的issues当中已经明确被标为bug了，具体的可以看[这里](https://github.com/lavas-project/vue-skeleton-webpack-plugin/issues/52)

### 参考链接
[Vue项目骨架屏注入实践](https://juejin.im/post/5b79a2786fb9a01a18267362) --- 介绍了大概的概念和几种注入方式  
[Vue项目骨架屏注入实践](https://segmentfault.com/a/1190000014832185) --- 腾讯老哥在思否的一篇文章，将服务端注入放到了前端来处理，使用了**vue-server-renderer**插件
[lavas的webpack相关](https://lavas.baidu.com/guide/v2/webpack/vue-skeleton-webpack-plugin) --- lavas是一套vue的pwa解决方案，但是我们只需要使用到相关的一个插件来完成我们想要的