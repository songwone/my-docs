# 状态更新流程

**触发更新的几种方式：**
* ReactDOM.render
* this.setState
* this.forceUpdate
* useState
* useReducer

## fiber双缓存
在React中最多会同时存在两棵Fiber树。当前屏幕上显示内容对应的Fiber树称为current Fiber树，正在内存中构建的Fiber树称为workInProgress Fiber树。
两个Fiber树的Fiber节点通过alternate相互引用。
> alternate: 代替者; 代理人; 候补者;

应用根节点(fiberRootNode)的current指针，指向current Fiber树

## 首次渲染

1. 首次执行ReactDOM.render  会创建fiberRootNode和rootFiber

2. 接下来进入render阶段，根据巨剑返回的JSX在内存中依次创建Fiber节点并构建Fiber树，被称为workInProgress Fiber树。
**在构建workInProgress Fiber树时，会尝试复用current Fiber树内的属性**
![123](/static/update-3.png)
