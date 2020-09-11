---
title: react-batch-update策略
date: 2020-09-11 16:19:48
categories: 
  - react
tags:
---

### 先思考两个问题

1. 一个function多次setstate
```javascript
export default function Parent() {
  let [count, setCount] = useState(0);

  function handleClick() {
    // 脱离react优化钩子，每次setstate都会触发render
    Promise.resolve().then((res) => {
      setCount(count + 1);
      setCount(count + 1);
    });
  }
  console.log("render component");
  return (
    <div onClick={handleClick}>
      Parent clicked {count} times
    </div>
  );
}
// 每次点击打印
// render component
// render component
// 触发两次
// 去除setTimeout
// 每次打印
// render component
// 触发一次
```
2. 事件点击
```javascript
  import React, { useState } from "react";
  import "./styles.css";
  export default function Parent() {
    let [count, setCount] = useState(0);

    function handleClick() {
      Promise.resolve().then((res) => {
        setCount(count + 1);
      });
    }
    console.log("render parent");
    return (
      <div onClick={handleClick}>
        Parent clicked {count} times
        <Child />
      </div>
    );
  }
  function Child() {
    let [count, setCount] = useState(0);

    function handleClick() {
      Promise.resolve().then((res) => {
        setCount(count + 1);
      });
    }

    console.log("render child");

    return (
      <button onClick={() => handleClick()}>Child clicked {count} times</button>
    );
  }
  // 每次点击打印
  // render child
  // render parent
  // render child

  // *** 进入 react click 的事件函数 ***
  // Child (onClick) 触发点击
  //   - setState 修改 state
  //   - re-render Child 重新渲染 //   不必要的
  // Parent (onClick) 触发点击（冒泡）
  //   - setState 修改 state
  //   - re-render Parent 重新渲染
  //   - re-render Child 重新渲染 （渲染是自顶向下的，父组件更新会导致子组件更新）
  // *** 退出 react click 的事件函数  ***

  // 去除setTimeout每次打印
  // render parent
  // render child
```

### react的做法
1. 每次setState都渲染是很浪费性能的，所以react做了batchUpdate
2. 通过isBatchingUpdates 标志，判断是不是可以batchupdate
3. react 合成事件（react对事件做过一次封装，我们成为合成事件），在合成事件里面，react会做一个wrapper，react可以知道什么时候开始、什么时候结束，会标记isBatchingUpdates为true，在事件结束时统一更新state
4. 同理react生命周期里面也会有这样的优化

### react做不到的优化
1. settimeout
2. promise
3. async function
4. ...
这些方法是异步的，react并不知道什么时候end（react wrapper方法已经执行完了，异步方法的callback为后面的event_loop的执行， react并没有对callback做wrapper，所以就没有isBatchingUpdates 标志位，每次setstate都会触发render）

### 异步中的解决方案
```javascript
import React, { useState } from "react";
import ReactDOM from "react-dom";

export default function Parent() {
  let [count, setCount] = useState(0);

  function handleClick() {
    Promise.resolve().then((res) => {
      ReactDOM.unstable_batchedUpdates(() => {
        setCount(count + 1);
        setCount(count + 1);
      });
    });
  }
  console.log("render component");
  return (
    <div onClick={handleClick}>
      Parent clicked {count} times
    </div>
  );
}
// 每次点击打印
// render component

```

### 参考文档
- [React State Batch Update](https://medium.com/swlh/react-state-batch-update-b1b61bd28cd2)
- [深入 react 细节之 - batchUpdate](https://zhuanlan.zhihu.com/p/78516581)
