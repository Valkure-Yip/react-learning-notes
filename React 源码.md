# React 源码



官方概览（有哪些模块）：[源码概览 – React](https://zh-hans.reactjs.org/docs/codebase-overview.html)



## React core 部分

core api 实现解析（较老react版本）[The React Source Code: a Beginner’s Walkthrough I | by Eric Churchill | Medium](https://medium.com/@ericchurchill/the-react-source-code-a-beginners-walkthrough-i-7240e86f3030)

官方文档：https://zh-hans.reactjs.org/docs/codebase-overview.html#react-core

官方推荐视频：构建一个简单的react core [Paul O Shannessy - Building React From Scratch - YouTube](https://www.youtube.com/watch?v=_MAD4Oly9yg)

## Fiber 架构 （virtual DOM）& Diffing

[React技术揭秘](https://react.iamkasong.com/#%E7%AB%A0%E8%8A%82%E5%88%97%E8%A1%A8)

Bookmark:https://react.iamkasong.com/diff/multi.html#%E5%A4%84%E7%90%86%E7%A7%BB%E5%8A%A8%E7%9A%84%E8%8A%82%E7%82%B9



### React 要解决的问题

我们日常使用App，浏览网页时，有两类场景会制约`快速响应`：

- 当遇到大计算量的操作或者设备性能不足使页面掉帧，导致卡顿。
- 发送网络请求后，由于需要等待数据返回才能进一步操作导致不能快速响应。

这两类场景可以概括为：

- CPU的瓶颈
- IO的瓶颈

#### CPU 瓶颈

JS可以操作DOM，`GUI渲染线程`与`JS线程`是互斥的。所以**JS脚本执行**和**浏览器布局、绘制**不能同时执行。

主流浏览器刷新频率为60Hz，即每（1000ms / 60Hz）16.6ms浏览器刷新一次。。当JS执行时间过长，超出了16.6ms，这次刷新就没有时间执行**样式布局**和**样式绘制**了。



**解决办法**：

在浏览器每一帧的时间中，预留一些时间给JS线程，`React`利用这部分时间更新组件（可以看到，在[源码 (opens new window)](https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/scheduler/src/forks/SchedulerHostConfig.default.js#L119)中，预留的初始时间是5ms）。

当预留的时间不够用时，`React`将线程控制权交还给浏览器使其有时间渲染UI，`React`则等待下一帧时间到来继续被中断的工作。

> 这种将长任务分拆到每一帧中，像蚂蚁搬家一样一次执行一小段任务的操作，被称为`时间切片`（time slice）



#### IO 瓶颈

网络延迟

`React`实现了[Suspense (opens new window)](https://zh-hans.reactjs.org/docs/concurrent-mode-suspense.html)功能及配套的`hook`——[useDeferredValue (opens new window)](https://zh-hans.reactjs.org/docs/concurrent-mode-reference.html#usedeferredvalue)。

而在源码内部，为了支持这些特性，同样需要将**同步的更新**变为**可中断的异步更新**



### React 15 （Old sturcture）

- Reconciler（协调器）—— 负责找出变化的组件

  JSX -> vdom; diffing

- Renderer（渲染器）—— 负责将变化的组件渲染到页面上

  ReactDom: vdom -> Dom



在**Reconciler**中，`mount`的组件会调用[mountComponent (opens new window)](https://github.com/facebook/react/blob/15-stable/src/renderers/dom/shared/ReactDOMComponent.js#L498)，`update`的组件会调用[updateComponent (opens new window)](https://github.com/facebook/react/blob/15-stable/src/renderers/dom/shared/ReactDOMComponent.js#L877)。这两个方法都会递归更新子组件。

由于**递归执行**，所以更新一旦开始，中途就无法中断。当层级很深时，递归更新时间超过了16ms，用户交互就会卡顿。



**Reconciler**和**Renderer**是交替工作的，当第一个`li`在页面上已经变化后，第二个`li`再进入**Reconciler**:

<img src="https://react.iamkasong.com/img/v15.png" alt="更新流程" style="zoom:50%;" />

#### React 16 (new)

- Scheduler（调度器）—— 调度任务的优先级，高优任务优先进入**Reconciler**
- Reconciler（协调器）—— 负责找出变化的组件
- Renderer（渲染器）—— 负责将变化的组件渲染到页面上





