---
title: snabbdom简单解读
copyright: true
date: 2019-04-01 10:55:17
tags:
- 前端
- 技术
categories: 技术
---

[snabbdom](https://github.com/snabbdom/snabbdom)是非常经典的一个库了。
曾经在知乎上看到有这样一个[问题](https://www.zhihu.com/question/29380608)，发现**virtual-dom**也是不少人都造过的轮子。
snabbdom的牛逼之处更多在于思想，而不是在于代码有多么的精妙，这一点在react和vue都用virtual-dom的思想去做dom的处理而又对其做了优化这件事情中可见一斑。
<!--more-->

### 前言
  1. 实际上在刚看到源码的时候也是有些一脸懵逼，所以就从[第一个commit](https://github.com/snabbdom/snabbdom/commits/master?after=81da0c124257d393e29796a13e92f6f3016ac20d+447)开始看了，确实不失为一个好方法，虽然可能花的时间会稍微长一点0.0，好处在于可以了解到作者最初的想法。  
  2. **diff**的部分实际上已经有非常非常多讲的很通俗的文章，写的辣鸡的地方可以评论补(peng)充(wo)啦，感谢！
  
### 项目结构
  先来看下项目结构
  ![项目结构](https://github.com/OctupleSakura/show-img/raw/master/blog/snabbdom/snabbdom-tree.jpg)
  modules文件夹里面都是拆分出来的简单替换和对比的一些函数，都比较简单可以自行查看。
  hero文件里面放了snabbdom的钩子函数，也就是hook，会在snabbom的init函数调用的时候收集，在对应的时候执行，同时里面通过**requestAnimationFrame**做了一些动画上的优化。
  其他比较核心的有   
  * h函数（传入对应参数生成vnode）
  * html（domapis一些dom-api封装）
  * snabbdoms.bundle（打包的一些处理）
  * snabbdom（主文件核心逻辑都在这里，主要分析这个里面的代码）
  * thunk（对patch的一些优化）
  * vnode（定义了virtual-dom的结构）    
  
    ```js
      export interface VNodeData {
        props?: Props;
        attrs?: Attrs;
        class?: Classes;
        style?: VNodeStyle;
        dataset?: Dataset;
        on?: On;
        hero?: Hero;
        attachData?: AttachData;
        hook?: Hooks;
        key?: Key;
        ns?: string; // for SVGs
        fn?: () => VNode; // for thunks
        args?: Array<any>; // for thunks
        [key: string]: any; // for any other 3rd party module
      }
    ```
  vnode的结构可以从定义的接口看出  
    
    
### 分析
[这段代码](https://github.com/snabbdom/snabbdom/blob/master/src/snabbdom.ts#L195)也就是我们常说的diff，也就是updateChildren的这个函数，接受了两个children数组来进行对比
```js
  function updateChildren(parentElm: Node,
                          oldCh: Array<VNode>,
                          newCh: Array<VNode>,
                          insertedVnodeQueue: VNodeQueue) {
    let oldStartIdx = 0, newStartIdx = 0;
    let oldEndIdx = oldCh.length - 1;
    let oldStartVnode = oldCh[0];
    let oldEndVnode = oldCh[oldEndIdx];
    let newEndIdx = newCh.length - 1;
    let newStartVnode = newCh[0];
    let newEndVnode = newCh[newEndIdx];
    let oldKeyToIdx: any;
    let idxInOld: number;
    let elmToMove: VNode;
    let before: any;

    //通过以上的几个变量来做到逐个的比较
    //主要在于oldStart和oldEnd的几个变量
    //通过封装好的sameVnode()的函数来对比，具体的可以自行查看
    //idx控制着每一次循环对比的元素，具体的变化看每一个判断规则所做的变化

    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
      if (oldStartVnode == null) { //没有oldStartVnode的时候我们把第一个oldCh的第一个Vnode保存起来
        oldStartVnode = oldCh[++oldStartIdx]; // Vnode might have been moved left
      } else if (oldEndVnode == null) { 
        oldEndVnode = oldCh[--oldEndIdx];
      } else if (newStartVnode == null) {
        newStartVnode = newCh[++newStartIdx];
      } else if (newEndVnode == null) { //到这位置基本都和上面类似
        newEndVnode = newCh[--newEndIdx];
      } else if (sameVnode(oldStartVnode, newStartVnode)) { //以上的判断都是设置好变量，这一个判断开始就是真正开始对比的逻辑，直接判断新旧vnode的头元素是否相等
        patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue);
        oldStartVnode = oldCh[++oldStartIdx];
        newStartVnode = newCh[++newStartIdx];
      } else if (sameVnode(oldEndVnode, newEndVnode)) { //同上，对比尾vnode
        patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue);
        oldEndVnode = oldCh[--oldEndIdx];
        newEndVnode = newCh[--newEndIdx];
      } else if (sameVnode(oldStartVnode, newEndVnode)) { //对比old头vnode和new尾vnode，相等说明元素移到了末尾
        patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue);
        api.insertBefore(parentElm, oldStartVnode.elm as Node, api.nextSibling(oldEndVnode.elm as Node));
        oldStartVnode = oldCh[++oldStartIdx];
        newEndVnode = newCh[--newEndIdx];
      } else if (sameVnode(oldEndVnode, newStartVnode)) { //基本同上，对比old尾vnode和new头vnode相同的话说明元素被移到了头部
        patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue);
        api.insertBefore(parentElm, oldEndVnode.elm as Node, oldStartVnode.elm as Node);
        oldEndVnode = oldCh[--oldEndIdx];
        newStartVnode = newCh[++newStartIdx];
      } else {

        //以上  在所有对比条件都不成立之后，说明基本上的对比逻辑基本走完，剩下的基本是不等的vnode
        // 通过createKeyToOldIdx创建一个json，key和index
        if (oldKeyToIdx === undefined) {
          oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx);
        }
        idxInOld = oldKeyToIdx[newStartVnode.key as string];
        //扔一个newStartVnode的key到json中，如果没有说明这个newStartVnode是新的节点，直接创建并且插到最前面
        if (isUndef(idxInOld)) { // New element
          api.insertBefore(parentElm, createElm(newStartVnode, insertedVnodeQueue), oldStartVnode.elm as Node);
          newStartVnode = newCh[++newStartIdx];
        } else {
        //如果在json中找到了对应的节点，说明已经有了变更
          elmToMove = oldCh[idxInOld];
          //这里需要判断一下sel是否相等，不等说明节点类型已经改变创建新节点
          if (elmToMove.sel !== newStartVnode.sel) {
            api.insertBefore(parentElm, createElm(newStartVnode, insertedVnodeQueue), oldStartVnode.elm as Node);
          } else {
            //如果节点类型相等的话就直接扔到patchVnode 对应修改一遍属性，并且移动到最前面
            patchVnode(elmToMove, newStartVnode, insertedVnodeQueue);
            oldCh[idxInOld] = undefined as any;
            api.insertBefore(parentElm, (elmToMove.elm as Node), oldStartVnode.elm as Node);
          }
          //那最后 我们每处理完一个newVnode就将idx+1
          newStartVnode = newCh[++newStartIdx];
        }
      }
    }
    
    //如果这里StartIdx依旧小于EndIdx的话， 说明有多余的节点
    if (oldStartIdx <= oldEndIdx || newStartIdx <= newEndIdx) {        
      //这一段主要用于删除掉或者添加进去多余或者缺失的节点
      //这一段说明 old节点已经处理完了，我们需要newVnode创建并添加
      if (oldStartIdx > oldEndIdx) {
        before = newCh[newEndIdx+1] == null ? null : newCh[newEndIdx+1].elm;
        addVnodes(parentElm, before, newCh, newStartIdx, newEndIdx, insertedVnodeQueue);
      } else {
        //这一段说明是new节点处理完成，直接删除多余节点
        removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx);
      }
    }
  }
```
以上把代码都做了一个注释，有兴趣的话可以拿张纸笔画画图会比较清晰(自己也是这么做的)。

### 一些疑惑和疏漏
1. 在查看比较前面的commit发现在最开始有用到**createDocumentFragment**这个api来避免浏览器多次渲染导致的回流和重绘，但发现后面给移除了..不太明白0.0
2. hook这个地方实际上有个很小的疑惑，在init的时候可以发现他的hook数组里面一般只有一个回调需要执行，为什么以数组的形式保存，可能是后面还有没写的hook？
3. patch的部分也可以细讲(懒了)
4. hook的几个函数没讲出来，之后再回头看看吧(咕咕咕)

### 水文结束
优化的细节实际上很多，阅读优秀的开源项目能收获到不少平时写代码的技巧。  
第一个commit十分简单，可以慢慢翻到补充稍微完整的源码之后再跳着看，这样可能会比较快。 
最近翻了翻pr，发现作者已经很久没有合并pr或者更新代码了，ci的test好像也有问题..  
原来是有了新欢[turbine](https://github.com/funkia/turbine)，好像也蛮有意思的，就是相关资料有点少，之后有空的话去看看啦~
<img src="https://github.com/OctupleSakura/show-img/raw/master/blog/snabbdom/sticker.webp" style="width:300px" > 