# Promise 解读

## 1. Promise 的含义

所谓 `Promise`，简单说其实是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。它是异步变成的一种解决方案，比传统的解决方案（回调函数和事件）更合理更强大。

`Promise` 是一个对象，从它可以获取异步操作的消息。有以下两个特点：

(1) 对象的状态不受外界影响。`Promise` 对象代表一个异步操作，有三种状态：`pending`（进行中）、`fulfilled`（已成功）和 `rejected`（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，其他任何操作都无法改变这个状态。
(2) 一旦状态发生改变，就不会再变，任何时候都可以得到这个结果。`Promise` 对象的状态改变，只有两种可能：从 `pending` 变为 `fulfilled` 和 从 `pending` 变为 `rejected` 。这两种情况一旦发生，就会一直保持这个结果，不会再变。如果状态改变已经发生了，再对 `Promise` 对象添加回调函数，也会立即得到这个结果。*这与事件(Event)完全不同，事件的特点是，如果你错过了，再去监听，是得不到结果的。*

有了 `Promise` 对象，就可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数。此外，`Promise` 对象提供统一的接口，使得控制异步操作更加容易。

`Promise` 的缺点。
i. 无法取消 `Promise` ，一旦新建它就会立即执行，无法中途取消；

ii. 如果不设置回调函数，`Promise` 内部抛出的错误，不会反应到外部；

iii. 当处于 `pending` 状态时，无法得知目前进展到拿一个阶段。

> 如果某些事件不断反复发生，一般来说，使用 `Stream` 模式是比部署 `Promise` 更好地选择。

----

## 2. Promise 的用法

```js
const promise = new Promise((resolve, reject) =>{
    //some code
    if (/*异步操作成功*/) {
        resolve(success)
    } else {
        reject(error)
    }
})
```

从上面代码可以看出， `Promise` 对象是一个构造函数，它会生成一个 `Promise` 实例。它节后一个函数作为参数，该函数的两个参数分别是 `resolve` 和 `reject`。

函数 `resolve` 和 `reject` 会将 `Promise` 对象的状态分别从“未完成”变为“成功”和“未完成”变为“失败”，在异步操作成功（失败）时调用，并将异步操作的结果，作为参数传递出去。


```js
promise.then(res => {
    // success
}, error => {
    // failure
})
```

观察上面代码可以看出，`Promise` 生成实例以后，可以用 `then` 方法分别指定 `resolved` 和 `rejected` 状态的回调函数。