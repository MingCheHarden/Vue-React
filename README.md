# Vue & React 数据流动及数据更新Dom渲染机制探究

背景：vue2.X、React v15.6.1
都是当前最新版
## Vue

整体结构是自上向下的单向数据流。
```
p.s. 子组件是不能修改props的，子组件虽然能通过emit改变父组件的状态，但实际数据流还是从父组件开始流向子组件，不用像React那样抽象一个最高级的父组件
```
但其实还是支持你在合适的场景使用双向绑定，v-model、.sync(2.3.0+)

数据更新监测机制是：基于响应式的数据依赖追踪。

#### 官方：

https://cn.vuejs.org/v2/guide/reactivity.html#如何追踪变化

#### 数据更新监测机制解读：
*  结合一段尤大的原话 https://www.zhihu.com/question/59953136
    ```
    对于这点，其实 Vue 的策略就是靠依赖追踪让组件自然成为更新的 boundary，这样就确保了永远只触发必要的组件更新，在组件内部则是 vdom diff，因为实际场景下一个或几个组件的 diff 几乎一定足够快。这样通过些微的性能来保留全局状态和 vdom 的好处。
    ```
    每个组件实例都有相应的 watcher 实例对象（可以在组件的watch选项中自定义watcher），它会在组件渲染的过程中把属性（组件中的data）记录为依赖，
    ```
    组件内的props会在哪里被watch？ 父组件or子组件内？我觉得是在父组件被watch，因为子组件不可以改变props，props被改变的地方只能在父组件。父组件改变状态，再由上而下单向流动。
    ```
    之后当依赖项的 setter 被调用时，会通知 watcher 重新计算，从而致使它关联的组件得以更新。

    当数据流动时：

        Data(change)--->Influenced Components--->Virtual Dom(diff)---> Real Dom


#### Vuex
完全是依赖于Vux现有的机制，借鉴Flux、Redux的公共状态管理方案。他跟Redux不同的点在，强耦合与Vue，不可独立使用，而Redux，是一套独立的公共状态管理方案。

-----

## React 

整体结构是自上向下的单向数据流。（比Vue更纯粹，根本不存在vue中子组件通过emit触发父组件更新状态的方法，官方建议你提前抽象好一个最高级父组件管理所有使用了公用状态的子组件。）

数据渲染监测机制是：shouldComponentUpdate+immutable

#### 官方

1.不可变数据 immutable
https://discountry.github.io/react/tutorial/tutorial.html#为什么不可变性在React当中非常重要

2.shouldComponentUpdate（SCU）https://discountry.github.io/react/docs/optimizing-performance.html#避免重复渲染

#### 数据更新监测机制解读：
```
在 React 应用中，当某个组件的状态发生变化时，它会以该组件为根，重新渲染整个组件子树
```
所以数据流动如下：

    Data(change)--->根据SCU，以改变状态的组件为根，去遍历整个组件子树找出是哪些组件需要重新渲染--->Virtual Dom(diff)---> Real Dom

其实相当于做了两次diff 首先在数据层，需要遍历找到哪些component要重新渲染，再映射到vdom，vdom到真实的dom又要diff去找哪些地方要修改。

p.s. 果断比vue的dom渲染效率低啊

所以若要开始使用React，首先就要开始谈优化：

* 本质就是想办法：在Data改变并触发SCU时减少消耗（与Vue相比，感觉好蠢啊）
    * 优化核心：immutable
    * 使用immutable优化Data改变效率。不然你就得像官方文档建议的每次深拷贝一个新的数据，然后再setState，等待视图更新。React框架本身不提供immutable数据类型支持。
    * 控制SCU，在组件内进行数据对比，若无变化便不更新相应组件的状态，减少消耗。这里使用immutable可提升数据比对效率，据说是0成本，┐(・o・)┌ 。

再多提一点：

immutable带给React的好处，主要是在第一步。React主要是通过让你修改State，最好是重新建立新的引用来让组件知道数据被改变，immutable可以让数据更多的共享底层的数据结构，提升效率，减少内存占用率。

还有... 时间旅行（没感觉到这个功能有卵用）


#### Redux
全局一个大的state，然后呢
```
newState = actions.reduce(reducer, initState)
```
嗯嗯，就这样....


## 总结
React、immutable、Redux，是互相独立的，但一套成熟的解决方案，这三项都不可或缺。

我感觉这也是造成React，学习曲线陡峭的原因，讲道理对新手不太友好。

而且因为他的数据渲染更新视图的核心机制是傻傻地从根组件遍历所有子组件进行数据对比，虽然immutable可帮助提升效率，但总觉得是因为核心机制不够高级而导致的hack处理方案。Vue的依赖追踪，能直接知道是哪些组件被更新，讲道理是核心机制上的超越，效率高是必然的结果。

况且Vue又上手快，那啥就...嗯嗯...是吧...⁄(⁄ ⁄•⁄ω⁄•⁄ ⁄)⁄

## 有关两者的一些讨论
*  dom渲染性能。

    必然是vue效率更高啊。
    https://cn.vuejs.org/v2/guide/comparison.html#性能

*  vdom的存在意义。

    首先我想到是React官网为React Native写的标语"Learn Once, Write Anywhere"。在内存维护一套virtualDom，最大的好处是说，思维不再受限制于浏览器运行时，你可以在多平台Node and power mobile apps，使用同套技术栈，你所关心的就是virtual Dom的状态，至于virtual Dom和Real Dom的映射关系，开发人员便可不用关心，实现跨端。

    但我想说的是virtual Dom的效率一定不如你手动操作真实的dom快，一定的⁄(⁄ ⁄•⁄ω⁄•⁄ ⁄)⁄