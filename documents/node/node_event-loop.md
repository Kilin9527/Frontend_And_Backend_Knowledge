<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

* [Node Event Loop 学习记录](#node-event-loop-学习记录)
	* [一、Node 架构](#一-node-架构)
	* [二、Event Loop](#二-event-loop)
		* [2.1. 什么是事件(Event)](#21-什么是事件event)
		* [2.2 什么是Event Loop](#22-什么是event-loop)
		* [2.3 阶段细节](#23-阶段细节)
			* [Timers阶段](#timers阶段)
			* [I/O(pending) Callbacks阶段](#iopending-callbacks阶段)
			* [poll阶段](#poll阶段)
			* [check阶段](#check阶段)
			* [close callbacks阶段](#close-callbacks阶段)
			* [更新Event Loop示意图](#更新event-loop示意图)
		* [2.4 setImmediate()和setTimeout()](#24-setimmediate和settimeout)
		* [2.5 process.nextTick()](#25-processnexttick)
			* [什么是process.nextTick()](#什么是processnexttick)
			* [process.nextTick()为什么会被允许存在，有什么用？](#processnexttick为什么会被允许存在有什么用)
		* [2.6 process.nextTick() 和 setImmediate()](#26-processnexttick-和-setimmediate)
		* [2.7为什么要使用process.nextTick()？](#27为什么要使用processnexttick)
	* [三、Node中的宏任务(MacroTask)和微任务(MicroTask)](#三-node中的宏任务macrotask和微任务microtask)
	* [四、总结](#四-总结)
	* [参考链接](#参考链接)

<!-- /code_chunk_output -->

# Node Event Loop 学习记录

## 一、Node 架构

Node.js 主要分为四大部分，Node Standard Library，Node Bindings，V8 引擎，Libuv(EventLoop)，架构图如下:

<!-- ![Node架构图](../../assets/images/node/Node_Common_1.png) -->
![Node架构图](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/assets/images/Node/Node_Common_1.png?raw=true)


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
### 2.1. 什么是事件(Event)
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

### 2.2 什么是Event Loop
Node是基于Javascript实现的，但是Javascript本身是单线程的，并不支持多线程，一旦遇到大量耗时任务阻塞了Javascript，那将是一场灾难。为了解决这个问题，Node使用了一种叫做Event Loop的技术，Node将这些耗时的，会阻塞Javascript线程的任务交由Event Loop来处理，当Event Loop处理完成之后，通知Javascript处理该事件对应的回调。这样，Node就拥有了多线程处理事件的能力。

Node将异步任务交给Libuv执行，Libuv本身是多线程的，可以将不同的异步任务放到不同的线程处理，在任务完成之后通知Javascript处理该任务对应的事件。
<!-- ![Node-Event-Loop](../../assets/images/node/Event-loop/Node_Event_Loop1.png) -->
![Node-Event-Loop](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/assets/images/Node/Event-loop/Node_Event_Loop1.png?raw=true)

当一个Node应用启动之后，它会初始化Event Loop，V8引擎开始解析并执行js代码，遇到异步任务，将其放入到Event Loop执行，Event Loop会根据异步任务的类型，分发到不同的Libuv线程中执行。

Event Loop分为六个阶段：
<!-- ![事件循环阶段示意图1](../../assets/images/node/Event-loop/Node_Event_Loop_Phases_1.png) -->
![事件循环阶段示意图1](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/assets/images/Node/Event-loop/Node_Event_Loop_Phases_1.png?raw=true)

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

### 2.3 阶段细节
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

**当event loop进入到poll阶段且timers队列中没有设置定时器**。会发生下列两种情况之一：
- 如果poll队列不为空，event loop会同步的迭代执行poll队列的回调，直到poll队列为空或者达到执行次数上限为止。
- 如果poll队列为空，会发生下列两种情况之一：
  - 如果设置了setImmediate()，event loop会结束poll阶段，进入check阶段，执行相应的回调。
  - 如果没有设置setImmediate()，event loop会等待其他的回调被添加到poll队列，新添加的回调会立即执行。

**当event loop进入到poll阶段且timers队列中设置了定时器**，那么event loop将会在poll队列处于空闲状态时，查找timers队列中到期的回调，如果有一个或多个定时器到期，event loop会回到timers阶段执行回调。

注:上文中提到 "*当event loop进入到poll阶段且timers队列中没有设置定时器...*"，官方文档并没有明确说明 "*当event loop进入到poll阶段且timers队列中设置了定时器*"会怎样，但是官方文档间接的描述了一下，上面那段是我按照官方文档自己整理出来的。
```
官方文档：
Once the poll queue is empty the event loop will check for timers whose time
thresholds have been reached. If one or more timers are ready, the event 
loop will wrap back to the timers phase to execute those timers' callbacks.
```

#### check阶段
一旦poll阶段完成，本阶段的队列中的回调会立即执行。
实际上，setImmediate()是一个特殊的定时器，它使用libuv的api来调度执行回调。
通常，随着代码的执行，event loop最终会进入poll阶段等待新事件的到来。但是，如果存在setImmediate()回调，并且poll处于空闲状态，poll会结束等待进入check阶段。

#### close callbacks阶段
如果一个socket或者handle突然关闭（比如：socket.destory()），close事件就会被提交到这个阶段。否则它将会通过process.nextTick()触发。

#### 更新Event Loop示意图
通过Event Loop阶段详情的学习，我们更新一下Event Loop的阶段结构图。
<!-- ![事件循环阶段示意图1](../../assets/images/node/Event-loop/Node_Event_Loop_Phases_2.png) -->
![事件循环阶段示意图1](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/assets/images/Node/Event-loop/Node_Event_Loop_Phases_2.png?raw=true)

### 2.4 setImmediate()和setTimeout()
setImmediate()和setTimeout()看起来比较相似，但是行为缺不相同，这取决于何时调用它们：
- setImmediate()是在poll阶段结束后，进入check阶段时调用。
- setTimeout()是在poll阶段空闲时，且timers中的定时器到期后调用。

来看一段代码：
```javascript {.line-numbers}
// timeout_vs_immediate.js
setTimeout(() => {
  console.log('timeout');
}, 0);

setImmediate(() => {
  console.log('immediate');
});
```
执行结果如下：
```javascript
$ node timeout_vs_immediate.js
timeout
immediate

$ node timeout_vs_immediate.js
immediate
timeout
```
如果在主模块调用这两个方法，二者的执行顺序是不确定的，看图：
<!-- ![timeout vs immediate 1](../../assets/images/node/Event-loop/Node_setTimeout_setImmediate_1.png) -->
![timeout vs immediate 1](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/assets/images/Node/Event-loop/Node_setTimeout_setImmediate_1.png?raw=true)
首先，Node源码中有个逻辑处理，setTimeout(fn, 0) => setTimeout(fn, 1)，也就是说，即便设置了timeout的最小等待时间为0ms，也会被node处理成等待1ms。

V8引擎解析执行js代码，将异步操作交由event loop处理，然后event loop将任务提交给cpu去执行。由于CPU是为整个操作系统服务器的，所以，CPU同时还可能在运行其他应用，在这样的条件下，就可能有两种情况发生：
**Case 1**：当event loop申请CPU执行代码的时候，CPU正在执行其他应用的任务，event loop需要等待其他任务执行完成，100ms过去，event loop获得CPU资源，执行Poll阶段回调，我们代码中没有其他异步任务，所以Poll队列为空，按照Poll阶段执行逻辑，发现timers中有到期的定时器，执行setTimeout的回调，然后进入Poll阶段，Poll进入空闲状态，发现有setImmediate，进入check阶段执行setImmediate的回调。所以，setTimeout先于setImmediate执行。
**Case 2**: 当event loop申请CPU执行代码的时候，CPU处于空闲状态，event loop立即获得CPU资源，执行Poll阶段代码，我们代码中没有其他异步任务，所以Poll队列为空，按照Poll阶段执行逻辑，检查timers中的定时器，发现没有到期的定时器，Poll进入空闲状态，发现有setImmediate，进入check阶段执行回调，执行完毕之后进入Poll阶段，等待其他I/O事件，并检查timers中的定时器是否到期，一旦timers中的定时器到期，立即执行setTimeout的回调。所以，setImmediate先于setTimeout执行。

再来看另一段代码：
```javascript {.line-numbers}
// timeout_vs_immediate.js
const fs = require('fs');

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});
```

执行结果如下：

```javascript
$ node timeout_vs_immediate.js
immediate
timeout

$ node timeout_vs_immediate.js
immediate
timeout
```
这段代码和上面唯一不同的地方就是setTimeout和setImmediate都是在一个异步回调中被解析执行，执行的过程看下图:
<!-- ![timeout vs immediate 2](../../assets/images/node/Event-loop/Node_setTimeout_setImmediate_2.png) -->
![timeout vs immediate 2](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/assets/images/Node/Event-loop/Node_setTimeout_setImmediate_2.png?raw=true)
解析:
- step1：V8解析执行readFile代码，由于readFile是一个异步函数，所以readFile的回调函数会被放入到Poll队列执行。
- step2：当代码执行到Poll阶段时，如果readFile已经完成，则会执行readFile的回调函数。
- step3：执行readFile的回调函数，将setTimeout放入到timers队列当中，将setImmediate放入到check阶段。
- step4：readFile的回调函数执行完毕，此时Poll队列为空，根据阶段详情中的[poll阶段](#poll阶段)的逻辑：当Poll队列为空时，查看设置了setImmediate，所以先进入check阶段，执行setImmediate。
- step5：执行完setImmediate，会进入timers阶段，然后执行setTimeout。
所以，在异步回调中设置的setImmediate永远会早于setTimeout执行。

### 2.5 process.nextTick()
#### 什么是process.nextTick()
* process.nextTick()是一个异步的API，参数接收一个回调函数。
* process.nextTick()不属于Event Loop的任何一部分，它有自己的队列：nextTickQueue。
* 在Event Loop的每个阶段完成之后，下一个阶段开始之前，都会检查一下nextTickQueue是否为空，如果不为空，node会顺序执行nextTickQueue里的每一个任务，直到清空nextTickQueue队列，Event Loop才会进入到下一个阶段。
* process.nextTick()允许递归调用，递归调用process.nextTick()任务会导致Event Loop无法进入到下一个阶段，无法处理其他I/O事件，从而阻塞Event Loop，这种情况被称为"starve" I/O。
#### process.nextTick()为什么会被允许存在，有什么用？
因为Node的设计理念：API应该始终是异步的即使有些地方是没必要的。
通过process.nextTick()可以保证回调总是在其他同步代码运行完成自后才会调用。
看个例子：
```javascript {.line-numbers}
let bar;

// this has an asynchronous signature, but calls callback synchronously
function someAsyncApiCall(callback) { callback(); }

// the callback is called before `someAsyncApiCall` completes.
someAsyncApiCall(() => {

  // since someAsyncApiCall has completed, bar hasn't been assigned any value
  console.log('bar', bar); // undefined

});

bar = 1;

// 输出结果： bar undefined
```
用户定义了一个异步签名的函数someAsyncApiCall()，但实际上操作是同步的。当它被调用时，其回调也在event loop中的同一阶段被调用了，因为someAsyncApiCall()实际上并没有任何异步动作。结果，在代码还没有执行到bar初始化的时候，就去访问变量bar，从而导致错误发生。

用process.nextTick()改造一下上面的代码：

```javascript {.line-numbers}
let bar;

function someAsyncApiCall(callback) {
  process.nextTick(callback);
}

someAsyncApiCall(() => {
  console.log('bar', bar);
});

bar = 1;

// 输出结果： bar 1
```
通过将callback放入到process.nextTick()中，脚本可以全部执行完成，使函数和变量先于回调函数执行。同时还能阻止event loop继续执行。

这是一个实际的例子：
```javascript {.line-numbers}
const server = net.createServer(() => {}).listen(8080);

server.on('listening', () => {});
```
当只有一个端口作为参数传入，端口会被立即绑定。所以监听回调可能被立即调用。问题是：on('listening') 回调在那时还没被注册。

为了解决这个问题，把listening事件加入到nextTick()队列，让脚本中的所有代码全部执行完，让用户可以设置任何他们需要的事件处理函数，然后再去执行listening事件。

*来看看另一个例子*：
```javascript {.line-numbers}
// dedup.js
const foo = [1, 2];
const bar = ['a', 'b'];

foo.forEach(num => {
  setImmediate(() => {
    console.log('setImmediate', num);
    bar.forEach(char => {
      process.nextTick(() => {
        console.log('process.nextTick', char);
      });
    });
  });
});
```
输出结果如下：
```javascript 
$ node dedup.js
setImmediate 1
setImmediate 2
process.nextTick a
process.nextTick b
process.nextTick a
process.nextTick b
```
* Step1：V8引擎解析并执行js代码，发现有两个setImmediate，放入Check队列中。
* Step2：当event loop进行到check阶段时，开始执行setImmediate1的回调。
* Step3：执行setImmediate1的回调，将两个nextTick添加到nextTickQueue中，此时nextTickQueue中有两个任务。
* Step4：执行setImmediate2的回调，将两个nextTick添加到nextTickQueue中，此时nextTickQueue中有四个任务。
* Step5：Check阶段结束。
* Step6：在进入下一个阶段前，清空nextTickQueue，依次执行nextTick任务。

### 2.6 process.nextTick() 和 setImmediate()
* process.nextTick()在Event Loop每一个阶段结束时调用。
* setImmediate()在进入到Check阶段时调用。

官方文档建议开发者在所有情况中使用setImmediate()，因为这可以让代码兼容更多的环境。

### 2.7为什么要使用process.nextTick()？
有两个主要原因：
1. 允许用户处理错误，回收用户不再使用的资源，或者可能尝试在下一个event loop继续执行之前重新发送请求等。
2. 有时需要允许回调*在调用栈展开之后，事件循环继续之前*运行。
看一下这个例子：
```javascript {.line-numbers}
const server = net.createServer();
server.on('connection', (conn) => { });

server.listen(8080);
server.on('listening', () => { });
```
假设listen()在event loop之前运行，但是listening事件的回调放置在setImmediate()中。当传递给listen()的参数是端口号，会立即绑定(如果参数是hostname，不会立即绑定)。
随着event loop的进行，必然后进入到poll阶段，这意味着存在一种可能：在listening事件之前触发了connection事件。

## 三、Node中的宏任务(MacroTask)和微任务(MicroTask)
宏任务(MacroTask)指的是第二章当中提到的异步回调，包括setTimeout，setInterval，setImmediate等。
微任务(MicroTask)在大部分情况下是指Promise的then回调。
在Node中，除了宏任务和微任务，还有一个独立于event loop之外的nextTickQueue。
这三者之间的关系如图：
<!-- ![宏任务，微任务和nextTickQueue](../../assets/images/node/event-loop/Node_Event_Loop_Phases_3.png) -->
![宏任务，微任务和nextTickQueue](https://github.com/Kilin9527/Frontend_And_Backend_Knowledge/blob/master/assets/images/Node/Event-loop/Node_Event_Loop_Phases_3.png?raw=true)
* 在每个event loop阶段之间，会清理nextTickQueue中的所有任务。
* 在每个event loop阶段之间，会清理微任务队列中的所有任务。
* nextTickQueue早于MicroTask队列执行。
* 如果在处理nextTickQueue过程中遇到新的nextTick，name新的nextTick会被添加到当前的nextTickQueue中继续执行。
* 基于上面第四条，不要递归调用nextTick，这样会导致I/O事件不能及时处理，发生I/O starvation现象。

来看一个例子：
```javascript {.line-numbers}
Promise.resolve().then(() => {
  console.log('resolve1');
});

process.nextTick(function() {
  console.log('tick1');
  process.nextTick(function() {
    console.log('tick2');
  });
  process.nextTick(function() {
    console.log('tick3');
  });
});

Promise.resolve().then(() => {
  console.log('resolve2');
});

process.nextTick(function() {
  console.log('tick4');
});


Promise.resolve().then(() => {
  console.log('resolve3');
});

process.nextTick(function() {
  console.log('tick5');
});
```
执行结果为：
```javascript
tick1, tick4, tick5, tick2, tick3, resolve1, resolve2, resolve3
```
执行顺序：
1. 解析并执行js代码。
2. js代码执行到第一行，将promise放入microtask队列。
3. js代码执行到第五行，将nexttick放入nextTickQueue。
4. js代码执行到第十五行，将promise放入microtask队列。
5. js代码执行到第十九行，将nexttick放入nextTickQueue。
6. js代码执行到第二十四行，将promise放入microtask队列。
7. js代码执行到第二十八行，将nexttick放入nextTickQueue。
8. 此时microtask队列有三个任务，nextTickQueue有三个任务，js代码执行完毕，开始event loop阶段。
9. nextTickQueue比microtask队列的优先级高，先执行nextTickQueue中的所有任务。
10. 输出tick1，将第一个nextTick回调中的两个新的nextTick添加到nextTickQueue中。继续执行nextTickQueue，分别输出tick4, tick5, tick2, tick3。nextTickQueue执行完毕。
11. 执行MicroTask队列中的任务，输出resolve1, resolve2, resolve3。

## 四、总结
* Node的高性能，非阻塞的特性是通过event loop实现的。
* event loop分为六个阶段，每个阶段都有自己的队列执行特定类型的任务。
* 在每个阶段过渡期间，会检查并清空nextTickQueue和MicroTask队列。
* Node中的宏任务和微任务。

## 参考链接
https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/
https://github.com/yjhjstz/deep-into-node/blob/master/chapter1/chapter1-0.md
https://segmentfault.com/a/1190000017893482#articleHeader4
https://segmentfault.com/a/1190000017920493#articleHeader10
https://www.jianshu.com/p/2a7ac1b3b382
https://www.cnblogs.com/yzfdjzwl/p/8182749.html
https://cnodejs.org/topic/57d68794cb6f605d360105bf