# Promise 解读

## 1. Promise 的含义

所谓 `Promise`，简单说其实是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。它是异步变成的一种解决方案，比传统的解决方案（回调函数和事件）更合理更强大。

`Promise` 是一个对象，从它可以获取异步操作的消息。有以下两个特点：

(1) 对象的状态不受外界影响。`Promise` 对象代表一个异步操作，有三种状态：`pending`（进行中）、`fulfilled`（已成功）和 `rejected`（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，其他任何操作都无法改变这个状态。
(2) 一旦状态发生改变，就不会再变，任何时候都可以得到这个结果。`Promise` 对象的状态改变，只有两种可能：从 `pending` 变为 `fulfilled` 和 从 `pending` 变为 `rejected` 。这两种情况一旦发生，就会一直保持这个结果，不会再变。如果状态改变已经发生了，再对 `Promise` 对象添加回调函数，也会立即得到这个结果。*这与事件(Event)完全不同，事件的特点是，如果你错过了，再去监听，是得不到结果的。*

有了 `Promise` 对象，就可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数。此外，`Promise` 对象提供统一的接口，使得控制异步操作更加容易。

`Promise` 的缺点:

* 无法取消 `Promise` ，一旦新建它就会立即执行，无法中途取消；
* 如果不设置回调函数，`Promise` 内部抛出的错误，不会反应到外部；
* 当处于 `pending` 状态时，无法得知目前进展到拿一个阶段。

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

* Promise 的执行顺序

```js
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});
promise.then(function() {
  console.log('resolved.');
});
console.log('Hi!');
```

上面的代码输出为

```bash
// Promise
// Hi!
// resolve
```

从上面代码可以看出，Promise 新建后会立即执行，所以 `Promise` 首先输出。然后是同步任务 `Hi!`，而 `then` 方法指定的回调函数，将在当前脚本所有同步任务执行完才会执行，因此 `resolved` 最后输出。

### 3. Promise 应用场景

1. 图片异步加载

```js
function loadImageAsync(function(resolve, reject)) {
    const img = new Image()
    img.onload = function(){
        resolve(img)
    }

    img.onrror = function(){
        reject(new Error('could not load image at' + url))
    }

    img.src = url
}
```

该代码中，使用 `Promise` 包装了一个图片加载的异步操作。如果加载成功，就调用 `resolve` 方法，否则调用 `reject` 方法。

2. Promise 用法

```js
const p1 = new Promise((resolve, reject) =>{
    // code1
})

const p2 = new Promise((resolve, reject) =>{
    //...
    resolve(p1)
})
```

观察上面代码， `p1` 和 `p2` 都是 Promise 的实例，但是 `p2` 的 `resove` 方法将 `p1` 作为参数，即一个异步操作的结果是返回另一个异步操作。

*注意，这时 `p1` 的状态就会传递给 `p2`，也就是说，`p1` 的状态决定了 `p2`的状态。如果`p1` 的状态是 `pending`, 那么 `p2` 的回调函数就会等待 `p1` 的状态改变；如果`p1` 的状态时 `resolved` 或者 `rejected`，那么`p2` 的回调函数就是立刻执行*

```js
const p1 = new Promise(function (resolve, reject) {
  setTimeout(() => reject(new Error('fail')), 3000)
})

const p2 = new Promise(function (resolve, reject) {
  setTimeout(() => resolve(p1), 1000)
})

p2
  .then(result => console.log(result))
  .catch(error => console.log(error))
// Error: fail
```

上面代码中，`p1`是一个 Promise，3秒之后变为 `rejected`。`p2`的状态在 1 秒之后改变，`resolve` 方法是返回 `p1`。由于 `p2` 返回的是另一个 Promise，导致 `p2` 自己的状态无效了，由 `p1` 的状态决定 `p2` 的状态。所以，后面的`then`语句都变成针对后者(p1)。又过了 2 秒，`p1` 变为 `rejected`，导致触发 `catch` 方法指定的回调函数。