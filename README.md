# Vue & React 数据流动核心机制概述
## Vue

整体结构是自上向下的单项数据流。

数据渲染监测机制是：基于响应式的双向绑定。

### 官方：

https://cn.vuejs.org/v2/guide/reactivity.html#如何追踪变化

解读：
1.  组件内data、watch选项，会被Vue使用 Object.defineProperty追踪依赖，在属性被访问和修改时通知变化。
    当数据流动时：
    
    组件内：
    Data(change)--->Influenced Components--->Virtual Dom(diff)---> Real Dom

    在结合一段yxx的原话 https://www.zhihu.com/question/59953136
    ```
    对于这点，其实 Vue 的策略就是靠依赖追踪让组件自然成为更新的 boundary，这样就确保了永远只触发必要的组件更新，在组件内部则是 vdom diff，因为实际场景下一个或几个组件的 diff 几乎一定足够快。这样通过些微的性能来保留全局状态和 vdom 的好处。
    ```
2.组件间的数据流动属于单向流动。

### Vuex
完全是依赖于Vux现有的机制，借鉴Flux、Redux的公共状态管理方案。

## React 

整体结构是自上向下的单项数据流。（同Vue）

数据渲染监测机制是：shouldComponentUpdate+immutable.js

### 官方

1.不可变数据 immutable
https://discountry.github.io/react/tutorial/tutorial.html#为什么不可变性在React当中非常重要

2.shouldComponentUpdate（SCU）https://discountry.github.io/react/docs/optimizing-performance.html#避免重复渲染

解读：
```
在 React 应用中，当某个组件的状态发生变化时，它会以该组件为根，重新渲染整个组件子树
```
Data(change)--->根据SCU，以改变状态的组件为根，去整个组件子树找出是哪些组件需要重新渲染--->Virtual Dom(diff)---> Real Dom

其实相当于做了两次diff 首先在数据层，需要遍历找到哪些component要重新渲染，再映射到vdom，vdom到真实的dom又要diff去找哪些地方要修改。

p.s. 果断比vue的dom渲染效率低啊

再多提一点：

immutable带给React的好处，主要是在第一步。React主要是通过让你修改State，最好是重新建立新的引用来让组件知道数据被改变，immutable可以让数据更多的共享底层的数据结构，提升效率，减少内存占用率。

还有... 时间旅行（没感觉到这个功能有卵用）


### Redux

//TODO
Redux带给了React啥？


## 有关两者的一些讨论
1.  dom渲染性能。

    必然是vue效率更高啊。
    https://cn.vuejs.org/v2/guide/comparison.html#性能

2.  vdom的存在意义。