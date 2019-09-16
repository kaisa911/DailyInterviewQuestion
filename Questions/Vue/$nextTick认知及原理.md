# nextTick 功能及原理

nextTick 是 vue 的核心功能，在 vue 的内部实现中也会经常用到 nextTick

## nextTick 功能

官方文档：在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。

> 2.1.0 起新增：如果没有提供回调且在支持 Promise 的环境中，则返回一个 Promise。请注意 Vue 不自带 Promise 的 polyfill，所以如果你的目标浏览器不是原生支持 Promise (IE：你们都看我干嘛)，你得自行 polyfill。

官方示例：

```js
// 修改数据
vm.msg = 'Hello';
// DOM 还没有更新
Vue.nextTick(function() {
  // DOM 更新了
});

// 作为一个 Promise 使用 (2.1.0 起新增，详见接下来的提示)
Vue.nextTick().then(function() {
  // DOM 更新了
});
```

可以看到，nextTick 主要功能就是改变数据后让回调函数作用于 dom 更新后。为什么需要在 dom 更新后再执行回调函数，我修改了数据后，不是 dom 自动就更新了吗？
这个和 JS 中的 Event Loop 有关，网上教程不计其数，在此就不再赘述了。
![](https://developer.mozilla.org/files/4617/default.svg)
函数调用形成了一个栈帧。对象被分配在一个堆中，即用以表示一大块非结构化的内存区域。一个 JavaScript 运行时包含了一个待处理的消息队列。每一个消息都关联着一个用以处理这个消息的函数。

举个实际的例子：
我们有个带有分页器的表格，每次翻页需要选中第一项。正常情况下，我们想的是点击翻页器，向后台获取数据，更新表格数据，操纵表格 API 选中第一项。
但是，你会发现，表格数据是更新了，但是并没有选中第一项。因为，你选中第一项时，虽然数据更新了，但是 DOM 并没有更新。此时，你可以使用 nextTick ，在 DOM 更新后再操纵表格第一项的选中。
那么，nextTick 到底做了什么了才能实现在 DOM 更新后执行回调函数？

## 源码分析

nextTick 的源码位于 `src/core/util/next-tick.js`，总计 118 行，十分的短小精悍，十分适合初次阅读源码的同学。
nextTick 源码主要分为两块：

- 能力检测
- 根据能力检测以不同方式执行回调队列

### 能力检测

这一块其实很简单，众所周知，Event Loop 分为宏任务（macro task）以及微任务（ micro task），不管执行宏任务还是微任务，完成后都会进入下一个 tick，并在两个 tick 之间执行 UI 渲染。
但是，宏任务耗费的时间是大于微任务的，所以在浏览器支持的情况下，优先使用微任务。如果浏览器不支持微任务，使用宏任务；但是，各种宏任务之间也有效率的不同，需要根据浏览器的支持情况，使用不同的宏任务。
nextTick 在能力检测这一块，就是遵循的这种思想。

```js
// Determine (macro) task defer implementation.
// Technically setImmediate should be the ideal choice, but it's only available
// in IE. The only polyfill that consistently queues the callback after all DOM
// events triggered in the same loop is by using MessageChannel.
/* istanbul ignore if */
// 如果浏览器不支持Promise，使用宏任务来执行nextTick回调函数队列
// 能力检测，测试浏览器是否支持原生的setImmediate（setImmediate只在IE中有效）
if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  // 如果支持，宏任务（ macro task）使用setImmediate
  macroTimerFunc = () => {
    setImmediate(flushCallbacks);
  };
  // 同上
} else if (
  typeof MessageChannel !== 'undefined' &&
  (isNative(MessageChannel) ||
    // PhantomJS
    MessageChannel.toString() === '[object MessageChannelConstructor]')
) {
  const channel = new MessageChannel();
  const port = channel.port2;
  channel.port1.onmessage = flushCallbacks;
  macroTimerFunc = () => {
    port.postMessage(1);
  };
} else {
  /* istanbul ignore next */
  // 都不支持的情况下，使用setTimeout
  macroTimerFunc = () => {
    setTimeout(flushCallbacks, 0);
  };
}
```

首先，检测浏览器是否支持 setImmediate，不支持就使用 MessageChannel，再不支持只能使用效率最差但是兼容性最好的 setTimeout 了。
之后，检测浏览器是否支持 Promise，如果支持，则使用 Promise 来执行回调函数队列，毕竟微任务速度大于宏任务。如果不支持的话，就只能使用宏任务来执行回调函数队列。

### 执行回调函数队列

```js
// 回调函数队列
const callbacks = []
// 异步锁
let pending = false

// 执行回调函数
function flushCallbacks () {
  // 重置异步锁
  pending = false
  // 防止出现nextTick中包含nextTick时出现问题，在执行回调函数队列前，提前复制备份，清空回调函数队列
  const copies = callbacks.slice(0)
  callbacks.length = 0
  // 执行回调函数队列
  for (let i = 0; i < copies.length; i++) {
    copies[i]()
  }
}

...

// 我们调用的nextTick函数
export function nextTick (cb?: Function, ctx?: Object) {
  let _resolve
  // 将回调函数推入回调队列
  callbacks.push(() => {
    if (cb) {
      try {
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      _resolve(ctx)
    }
  })
  // 如果异步锁未锁上，锁上异步锁，调用异步函数，准备等同步函数执行完后，就开始执行回调函数队列
  if (!pending) {
    pending = true
    if (useMacroTask) {
      macroTimerFunc()
    } else {
      microTimerFunc()
    }
  }
  // $flow-disable-line
  // 2.1.0新增，如果没有提供回调，并且支持Promise，返回一个Promise
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}
```

总体流程就是：接收回调函数，将回调函数推入回调函数队列中。
同时，在接收第一个回调函数时，执行能力检测中对应的异步方法（异步方法中调用了回调函数队列）。
如何保证只在接收第一个回调函数时执行异步方法？
nextTick 源码中使用了一个异步锁的概念，即接收第一个回调函数时，先关上锁，执行异步方法。此时，浏览器处于等待执行完同步代码就执行异步代码的情况。
打个比喻：相当于一群旅客准备上车，当第一个旅客上车的时候，车开始发动，准备出发，等到所有旅客都上车后，就可以正式开车了。
当然执行 flushCallbacks 函数时有个难以理解的点，即：为什么需要备份回调函数队列？执行的也是备份的回调函数队列？
因为，会出现这么一种情况：nextTick 的回调函数中还使用 nextTick。如果 flushCallbacks 不做特殊处理，直接循环执行回调函数，会导致里面 nextTick 中的回调函数会进入回调队列。这就相当于，下一个班车的旅客上了上一个班车。

### 实现一个简易的 nextTick

```js
let callbacks = [];
let pending = false;

function nextTick(cb) {
  callbacks.push(cb);

  if (!pending) {
    pending = true;
    setTimeout(flushCallback, 0);
  }
}

function flushCallback() {
  pending = false;
  let copies = callbacks.slice();
  callbacks.length = 0;
  copies.forEach(copy => {
    copy();
  });
}
```

可以看到，在简易版的 nextTick 中，通过 nextTick 接收回调函数，通过 setTimeout 来异步执行回调函数。通过这种方式，可以实现在下一个 tick 中执行回调函数，即在 UI 重新渲染后执行回调函数。
