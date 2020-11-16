### 一. Node.js 是什么

Node.js 不是编程语言，不是一个 Web 框架，Node.js 是一个平台，它可以让 JavaScript 调用操作系统提供的接口，让 JavaScript 也能开发后端应用。

Node.js 技术架构图：

![](/madao.github.io/database/images/articles/node/basis/image.png)

- Node.js API: Node.js 内置的标准库
- Node.js bindings: js 代码和 c/c++代码沟通的桥梁，比如在 Node.js 中可以读写文件，`fs.readFile()`，但是 js 代码并没有读写文件的能力，所以调用`fs.readFile()`实际上是 c++代码去完成这个操作，从 js 代码调用，然后 c++代码响应并执行，中间的通道就是 bindings，其实在浏览器环境只能够也存在 bindings，比如：`document.querySelector()`，它并不是 js 提供的功能，它由浏览器提供。
- c/c++ addons：c++插件，Node.js 提供插件功能，可以让原生的 c++代码编译成`.node`文件，这样在 Node.js 中，可以通过 require 直接引用
- v8：大名鼎鼎的 js 引擎，在 Node.js 中提供 js 的运行环境
- libuv：一个“跨平台”的异步 I/O 库
- c-ares、http_parser...：一些使用 c++编写的库（目前还不是很了解这些库）

### 二. Node.js 工作流程

![](/madao.github.io/database/images/articles/node/basis/image1.jpeg)

[图片来源](https://mobile.twitter.com/BusyRich/status/494959181871316992)

尝试这理解这张工作流程的图片：

- application：开发者使用 js 写的程序
- v8：js 引擎
- node.js bindings：上面解释过，js 代码和 c++代码通信的桥梁
- libuv: 跨平台的异步 io 库，但是张图比技术架构那张图要详细一些，这里面的 Event Loop 是很重要的一个概念，后面会详细解释到

根据图中的箭头，可以梳理出一个流程：
Node.js 将 js 代码交给 v8 引擎去执行，js 代码会有一些异步任务，比如文件读写，发起网络请求，定时器等等，Node.js 遇到这种代码，会通过 bindings 将这些异步任务交给 libuv 去处理，libuv 通过 Event Loop 机制来处理异步任务，libuv 将异步任务交给其他线程去执行，当异步任务完成后，会调用该任务注册的回调函数（通过 bindings 可以在 c++代码中执行 js 代码），这样 js 代码就可以获取到异步任务执行的结果了。

有一个疑问，异步任务是有顺序的吗？

比如：一个读取文件的异步任务和一个定时器任务同时完成，那么是先执行读取文件异步任务的回调函数还是先执行定时器任务的回调函数，答案是异步任务是有优先级的，因为 JS 执行是单线程，不能同时执行两个函数。

决定异步任务顺序的就是 Event Loop，下面看看 Event Loop 是什么

### 三. Event Loop

[官方文档解释](https://nodejs.org/zh-cn/docs/guides/event-loop-timers-and-nexttick/)

简单解释下：字面意思是“事件循环”，不断的等待事件发生，当某个事件触发时，将这个事件的处理函数（回调函数）按照，订阅该事件的顺序依次执行，当这些处理函数执行完毕后，继续等待下一个事件触发，一直循环。

在 Node.js 中 Event Loop 分为 6 个阶段：

![](/madao.github.io/database/images/articles/node/basis/image2.png)

每个阶段都有一个先进先出（FIFO）的队列来执行事件的回调，直到队列为空，或者执行的回调数量达到最大值，Event Loop 将移至下一个阶段，来看下每个阶段具体都做些什么：

- timers：执行 setTimeout 或者 setInterval 注册的回调函数

- pending callbacks

  执行延迟到下一个循环迭代的 I/O 回调

  官网的解释有点难以理解，我是看了别人的文章理解的（[原文地址](https://set.sh/post/200317-how-nodejs-event-loop-works)），但是不知道对不对，一般 I/O 相关的回调函数都在 poll 阶段执行，但是有时会存在一些被延迟执行的 I/O 回调函数，这个阶段就是用于执行这些被延迟执行的 I/O 回调函数。

  由于官网也没有例子，这片博客也没有举例说明，所以我对被延迟的 I/O 回调函数还是有点难以理解，按照文档上说的每个阶段都会将队列中的回调执行完毕或者执行的回调数量达到最大值才会进入下一个阶段，我结合这段描述理解的就是，在 poll 阶段最大值执行次数之外的那些 I/O 回调函数，会被延迟到下一次循环的 pending callbacks 阶段，或者说这是 libuv 在“某些情况”下主动的去延迟调用，但是“某些情况”又是什么呢？

- idle, prepare：仅供内部使用
- poll

  检索（等待）新的 I/O 事件，并执行 I/O 相关的回调函数，适当情况下 Node.js 会阻塞在这个阶段

  文档总是说的很难理解，适当情况的情况又是什么情况呢？

  我的理解是这样的，比如：

  ```
  setTimeout(() => {
    console.log(1)
  }, 100)
  ```

  现在有一个定时器，100ms 后会执行，除此之外没有任何其他的异步任务了，那么 Event Loop 就停留在这个阶段，等待定时器时间到了，绕回 timers 阶段执行这个定时器的回调函数。

  注意：**poll 阶段停留的最大时间是有限制的，具体时间因操作系统而异。假如在停留的时候有一个 I/O 相关的回调函数被放进 poll 阶段的队列中，那么会先执行 poll 队列中的回调函数，再绕去（从 check 阶段方向回到 timers） timers 阶段执行定时器相关的回调。**

  所以如果 poll 阶段的回调函数执行的时间超过定时器设置的时间，那么定时器的回调函数执行时间就会延后

  总结一下这个阶段做的事情：

  1. 计算阻塞的时间

     - 如果 poll 队列不为空，那么会执行 poll 队列中的回调函数，知道队列被清空或者到达最大的停留时间

     - 如果 poll 队列为空：

       - 如果有 setImmediate 的回调函数，结束当前阶段，进入 check 阶段，执行 setImmediate 的回调函数
       - 如果没有 setImmediate 的回调函数，等待新的 I/O 回调被添加到队列中，并执行它
       - 检测是否有定时器到期，如果有定时器到期，绕回 timers 阶段执行到期的定时器回调

  2. 执行 poll 队列中的回调函数

- check：执行 setImmediate 注册的回调函数
- close callbacks: 一些关闭相关的回调函数在此阶段执行，比如：
  `socket.on('close', ...)`，我暂时没有接触到这些事件，所以照搬官网的解释

我觉得结合 libuv 的文档可以更好的理解，[文档地址](http://docs.libuv.org/en/latest/design.html)

### 四. setImmediate 和 setTimeout

从 Event Loop 的阶段来看，如果：

```
setTimeout(() => {
  console.log('setTimeout')
}, 0)

setImmediate(() => {
  console.log('setImmediate')
})
```

这样的代码 setTimeout 一定比 setImmediate 先执行，但实际上并不是这样：

![](/madao.github.io/database/images/articles/node/basis/image3.png)

这种情况下 setTimeout 和 setImmediate 的执行顺序是不确定的，官方文档解释了原因，原因是：

> 执行计时器的顺序将根据调用它们的上下文而异。如果二者都从主模块内调用，则计时器将受进程性能的约束（这可能会受到计算机上其他正在运行应用程序的影响）。
> 例如，如果运行以下不在 I/O 周期（即主模块）内的脚本，则执行两个计时器的顺序是非确定性的，因为它受进程性能的约束

又是这样，文档也不说清楚“受进程性能的约束”是什么意思，好在 google 编程大法给力，在 stackoverflow 上找到了这样一个问题[why is the nodejs event loop non-deterministic after the first run?](https://stackoverflow.com/questions/43566082/why-is-the-nodejs-event-loop-non-deterministic-after-the-first-run)

这里面有解释到：

libuv 启动一个事件循环的函数是`uv_run`，`uv_run`有一个函数调用`uv__update_time(loop)`，当这个函数执行完了之后就开始依次执行这次事件循环中的阶段

![](/madao.github.io/database/images/articles/node/basis/image4.png)

从函数名就能看出来，从`uv__update_time(loop)`一路看下去，最终调用了一个函数`clock_gettime`（在 unix 平台下）, 受进程性能的约束指的就是这个函数，原回答中说到它受计算机上运行的其他应用程序影响，所以最终的原因是：

`setTimeout` 的最小延迟时间是 1，这个[文档](https://nodejs.org/dist/latest-v15.x/docs/api/timers.html#timers_settimeout_callback_delay_args)有说明，所以：`setTimeout(fn, 0)` 等于 `setTimeout(fn, 1)`，Event Loop 初始化的时候，进入 timers 阶段前需要调用`uv__update_time`，`uv__update_time`的执行时间受到进程性能的影响，当进入 timers 阶段的时间小于 1ms，那么 setTimeout 的回调就不会放进 timers 的队列中，然后 Event Loop 进入下面的阶段，一直到 check 阶段，那么 setImmediate 就先执行，再绕回 timer 阶段执行 setTimeout，如果进入 timers 阶段的时间大于 1ms，那么 setTimeout 的回调就会被放进 timers 阶段的队列中，因为定时器到期了，这样 setTimeout 就先执行，setImmediate 后执行

假如 setTimeout 和 setImmediate 在同一个异步回调周期内，比如这样：

```
setTimeout(() => {
  setTimeout(() => {
    console.log('setTimeout')
  }, 0)

  setImmediate(() => {
    console.log('setImmediate')
  })
})
```

setImmediate 一定会在 setTimeout 之前执行

### 五. process.nextTick

除了上面说的那些异步任务之外，Node.js 还提供了一种异步任务：`process.nextTick`，上面的 Event Loop 的每个阶段都没有说到它，它是在当前阶段进入下一个阶段之前执行了，比如：poll 到 check 阶段，顺序是这样的：`poll -> process.nextTick -> check`。

但其实除了`process.nextTick`，还有 Promise，经过测试 Promise 和`process.nextTick`都会在阶段切换的时候执行，`process.nextTick`优先级更高，先`process.nextTick`再 Promise，这里说的 Promise 指的是 Promise 的 then 部分
。