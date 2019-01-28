<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [Node Event Loop 学习记录](#node-event-loop-学习记录)
	* [一、Node 架构](#一-node-架构)
	* [二、Event Loop](#二-event-loop)
		* [1. 什么是事件(Event)](#1-什么是事件event)
		* [2. 什么是Event Loop](#2-什么是event-loop)
		* [3. 阶段细节](#3-阶段细节)
			* [Timers阶段](#timers阶段)
			* [I/O(pending) Callbacks阶段](#iopending-callbacks阶段)
			* [poll阶段](#poll阶段)
			* [check阶段](#check阶段)
			* [close callbacks阶段](#close-callbacks阶段)
	* [参考链接](#参考链接)

<!-- /code_chunk_output -->

# Node Event Loop 学习记录

## 一、Node 架构

Node.js 主要分为四大部分，Node Standard Library，Node Bindings，V8 引擎，Libuv(EventLoop)，架构图如下:

![Node架构图](./assets/images/Node/Node_Common_1.png)

- **Node Standard Library**: Node提供的基础类库，例如http,fs,buffer,net等。
- **Node Bindings**: 负责JS与C++代码的沟通，封装V8引擎和Libuv库，给Node Standard Library提供API支持。
- **Node底层Node主要运行层，本层是由C/C++代码实现**
  - **V8引擎**: 由Google开发维护的Javascript引擎，负责解析执行Javascript代码。
  - **Libuv**: Libuv是一个高性能的，事件驱动的I/O库，提供跨平台抽象层，针对不同的操作系统封装，提供统一的API。Node的异步非阻塞I/O实际上是由Libuv实现的。
  - **C-ares**: 提供异步处理DNS的能力。
  - **Http_parser,OpenSSLz,lib等**: 提供包括 http 解析、SSL、数据压缩等其他的能力。

**Node运行顺序如下**：
1. V8引擎解析JS代码。
2. 解析后的代码调用Node API。
3. Libuv负责Node API执行，将不同的任务放入不同的线程，形成一个Event Loop，并以异步的方式将任务的执行结构返回给V8引擎。
4. V8引擎将结果返回给用户。

本文主要目的是为了理清**Event Loop阶段**(也就是上面介绍的第三阶段)发生的事情。


## 二、Event Loop
### 1. 什么是事件(Event)
事件是一种可以被侦测到的行为，并且允许用户自定义处理该事件。例如：
```javascript {.line-numbers}
// 设置一个定时器，在指定时间之后执行某个操作。
setTimeout(() => {
    console.log("Fired");
}, 1000);
```
```javascript {.line-numbers}
// 异步操作Http请求完成。
Promise.then(() => {
    console.log("Http request completed");
});

// 异步操作读取文件完成。
fs.readFile(path, () => {
    console.log("Read file completed");
})
```
可以观察到所有事件的处理都是在回调函数中进行的，所以，***Event Loop主要处理任务是管理这些回调函数的执行顺序***，理解Event Loop可以帮助我们写出更加可靠的代码。

### 2. 什么是Event Loop
Node是基于Javascript实现的，但是Javascript本身是单线程的，并不支持多线程，一旦遇到大量耗时任务阻塞了Javascript，那将是一场灾难。为了解决这个问题，Node使用了一种叫做Event Loop的技术，Node将这些耗时的，会阻塞Javascript线程的任务交由Event Loop来处理，当Event Loop处理完成之后，通知Javascript处理该事件对应的回调。这样，Node就拥有了多线程处理事件的能力。

Node将异步任务交给Libuv执行，Libuv本身是多线程的，可以将不同的异步任务放到不同的线程处理，在任务完成之后通知Javascript处理该任务对应的事件。

![](./assets/images/Node/Event-loop/Node_Event_Loop1.png)

当一个Node应用启动之后，它会初始化Event Loop，V8引擎开始解析并执行js代码，遇到异步任务，将其放入到Event Loop执行，Event Loop会根据异步任务的类型，分发到不同的Libuv线程中执行。

Event Loop分为六个阶段：
```
   ┌────────────────────────────────┐
┌─>│        timers                  │
│  └──────────┬─────────────────────┘
│  ┌──────────┴─────────────────────┐
│  │     I/O(pending) callbacks     │
│  └──────────┬─────────────────────┘
│  ┌──────────┴─────────────────────┐
│  │     idle, prepare              │
│  └──────────┬─────────────────────┘      ┌───────────────┐
│  ┌──────────┴─────────────────────┐      │   incoming:   │
│  │         poll                   │<─────┤  connections, │
│  └──────────┬─────────────────────┘      │   data, etc.  │
│  ┌──────────┴─────────────────────┐      └───────────────┘
│  │        check                   │
│  └──────────┬─────────────────────┘
│  ┌──────────┴─────────────────────┐
└──┤    close callbacks             │
   └────────────────────────────────┘
```
每个阶段都有一个执行callback的FIFO队列，直到队列中的callback被清空或达到最多执行次数限制之后，才会进入下一个阶段。

**阶段概述：**
1. **timers**：执行setTimeout和setInterval的回调。
2. **I/O(pending) Callbacks**：处理一些上一轮循环中的少数未执行的 I/O 回调。
3. **idle, prepare**：Node内部使用。
4. **poll**：
    4.1 检查新的I/O事件。
    4.2 执行除了close callbacks，定时器回调和setImmediate之外的所有回调。
    4.3 Node会在适当的时候阻塞。
5. **check**：执行setImeediate的回调。
6. **close callbacks**：一些close callbacks。例如socket.on('close', ...)。

### 3. 阶段细节
#### Timers阶段 
定时器会指定执行回调的等待时间，一旦到期，timers会尽可能早一点执行定时器的回调，这个“早”是相对的，取决于操作系统当前执行的任务，分两种情况：
*假设定时器设置的等待时间为x*
1.如果到达定时器的等待时间，操作系统中没有其他任务在执行，可以立即执行定时器指定的回调。*实际等待时间为x*。
2.如果到达定时器的等待时间，操作系统有其他任务正在执行，那么timers队列需要等待操作系统的其他任务执行完成之后，才可以开始执行timers中的回调，这时候实际等待的时间就会比预设的时间长。*实际等待时间为x+n*。
所以，timers只能保证尽可能的早去执行定时器的回调。
```
Note：严格意义上说，是Poll阶段决定了什么时候执行timers阶段的内容。
```

#### I/O(pending) Callbacks阶段
该阶段执行一些系统的回调，例如TCP错误事件。

#### poll阶段
poll阶段有两个主要功能：
1. 当timers中的定时器到时后，执行指定的回调。
2. 处理poll队列中的事件。

当进入到poll阶段，timers队列中没有设置定时器。会发生下列两种情况之一：
- 如果poll队列不为空，event loop会同步的迭代执行poll队列的回调，直到poll队列为空或者达到执行次数上限为止。
- 如果poll队列为空，会发生下列两种情况之一：
  - 如果设置了setImmediate()，event loop会结束poll阶段，进入check阶段，执行相应的回调。
  - 如果没有设置setImmediate()，event loop会等待其他的回调被添加到poll队列，新添加的回调会立即执行。

一旦poll队列为空，event loop将在timers队列中寻找到期的定时器，如果有一个或多个定时器到期，event loop会回到timers阶段执行到期的回调。

#### check阶段
一旦poll阶段完成，本阶段的队列中的回调会立即执行。
实际上，setImmediate()是一个特殊的定时器，它使用libuv的api来调度执行回调。
通常，随着代码的执行，event loop最终会进入poll阶段等待新事件的到来。但是，如果存在setImmediate()回调，并且poll处于空闲状态，poll会结束等待进入check阶段。

#### close callbacks阶段
如果一个socket或者handle突然关闭（比如：socket.destory()），close事件就会被提交到这个阶段。否则它将会通过process.nextTick()触发。


```javascript {.line-numbers}
const fs = require('fs');

function someAsyncOperation(callback) {
  // Assume this takes 95ms to complete
  fs.readFile('/path/to/file', callback);
}

const timeoutScheduled = Date.now();

setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;

  console.log(`${delay}ms have passed since I was scheduled`);
}, 100);


// do someAsyncOperation which takes 95 ms to complete
someAsyncOperation(() => {
  const startCallback = Date.now();

  // do something that will take 10ms...
  while (Date.now() - startCallback < 10) {
    // do nothing
  }
});
// 111ms have passed since I was scheduled
```
## 参考链接
https://github.com/yjhjstz/deep-into-node/blob/master/chapter1/chapter1-0.md
https://segmentfault.com/a/1190000017893482#articleHeader4