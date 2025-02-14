#### event loop

js 是单线程的，所有任务需要排队，前一个任务结束，才会执行后一个任务。如果前一个任务耗时很长，后一个任务就不得不一直等着。但是IO设备（输入输出设备）很慢（比如Ajax操作从网络读取数据），js 不可能等待IO设备执行完成才继续执行下一个的任务，这样就失去了这门语言的意义。所以 js 的任务分为同步任务和异步任务。

1.所有同步任务都是在主线程执行，形成一个“执行栈”（就是下图中的stack）；

2.所有的异步任务都会暂时挂起，等待运行有了结果之后，其回调函数就会进入“任务队列”（task queue）排队等待；

3.当执行栈中的所有同步任务都执行完成之后，就会读取任务队列中的第一个的回调函数，并将该回调函数推入执行栈开始执行；

4.主线程不断循环重复第三步，这就是“event loop”的运行机制。

![img](https://user-gold-cdn.xitu.io/2019/8/11/16c7e63e97a272a6?imageslim)

上图中，主线程运行的时候，产生堆（heap）和栈（stack），堆用来存放数组对象等引用类型，栈中的代码调用各种外部API，它们在"任务队列"中加入各种事件（click，load，done）。只要栈中的代码执行完毕，主线程就会去读取"任务队列"，依次执行那些事件所对应的回调函数。

###### 任务队列中有两种异步任务，一种是宏任务，包括`script setTimeout setInterval`等，另一种是微任务，包括`Promise process.nextTick MutationObserver`等。每当一个 js 脚本运行的时候，都会先执行`script`中的整体代码；当执行栈中的同步任务执行完毕，就会执行微任务中的第一个任务并推入执行栈执行，当执行栈为空，则再次读取执行微任务，循环重复直到微任务列表为空。等到微任务列表为空，才会读取宏任务中的第一个任务并推入执行栈执行，当执行栈为空则再读取执行微任务，微任务为空才再读取执行宏任务，如此循环。

##### This指向

优先级：

> ```
> fn() < obj.fn() < fn.call(obj) < new fn()
> ```

首先，`new`调用的优先级最高，只要有`new`关键字，`this`就指向实例本身；接下来如果没有`new`关键字，有`call、apply、bind`函数，那么`this`就指向第一个参数；然后如果没有`new、call、apply、bind`，只有`obj.foo()`这种点调用方式，`this`指向点前面的对象；最后是光杆司令`foo()` 这种调用方式，`this`指向`window`（严格模式下是`undefined`）。

es6中新增了箭头函数，而箭头函数最大的特色就是没有自己的`this、arguments、super、new.target`，并且箭头函数没有原型对象`prototype`不能用作构造函数（`new`一个箭头函数会报错）。**因为没有自己的this，所以箭头函数中的this其实指的是包含函数中的this**。无论是点调用，还是`call`调用，都无法改变箭头函数中的`this`。

##### 闭包

闭包其实是一种特殊的函数，它可以访问函数内部的变量，还可以让这些变量的值始终保持在内存中，不会在函数调用后被垃圾回收机制清除。
JS是`单线程`的，JS是通过事件队列`(Event Loop)`的方式来实现异步回调的。

##### CPU

计算机的核心是`CPU`，它承担了所有的计算任务。

它就像一座工厂，时刻在运行。

##### 进程

假定工厂的电力有限，一次只能供给一个车间使用。 也就是说，一个车间开工的时候，其他车间都必须停工。 背后的含义就是，单个CPU一次只能运行一个任务。

```
进程`就好比工厂的车间，它代表CPU所能处理的单个任务。 `进程`之间相互独立，任一时刻，CPU总是运行一个`进程`，其他`进程`处于非运行状态。 CPU使用时间片轮转进度算法来实现同时运行多个`进程
```

##### 线程

一个车间里，可以有很多工人，共享车间所有的资源，他们协同完成一个任务。

`线程`就好比车间里的工人，一个`进程`可以包括多个`线程`，多个`线程`共享`进程`资源。

##### CPU、进程、线程之间的关系

从上文我们已经简单了解了CPU、进程、线程，简单汇总一下。

- `进程`是cpu资源分配的最小单位（是能拥有资源和独立运行的最小单位）
- `线程`是cpu调度的最小单位（线程是建立在进程的基础上的一次程序运行单位，一个进程中可以有多个线程）
- 不同`进程`之间也可以通信，不过代价较大
- `单线程`与`多线程`，都是指在一个`进程`内的单和多

##### 浏览器是多进程的

我们已经知道了`CPU`、`进程`、`线程`之间的关系，对于计算机来说，每一个应用程序都是一个`进程`， 而每一个应用程序都会分别有很多的功能模块，这些功能模块实际上是通过`子进程`来实现的。 对于这种`子进程`的扩展方式，我们可以称这个应用程序是`多进程`的。

总结一下：

- 浏览器是多进程的
- 每一个Tab页，就是一个独立的进程

##### 浏览器包含了哪些进程

- 主进程 

  - 协调控制其他子进程（创建、销毁）
  - 浏览器界面显示，用户交互，前进、后退、收藏
  - 将渲染进程得到的内存中的Bitmap，绘制到用户界面上
  - 处理不可见操作，网络请求，文件访问等

- 第三方插件进程 

  - 每种类型的插件对应一个进程，仅当使用该插件时才创建

- GPU进程 

  - 用于3D绘制等

- ```
  渲染进程
  ```

  ，就是我们说的

  ```
  浏览器内核
  ```

  - 负责页面渲染，脚本执行，事件处理等
  - 每个tab页一个渲染进程

那么浏览器中包含了这么多的进程，那么对于普通的前端操作来说，最重要的是什么呢？

答案是`渲染进程`，也就是我们常说的`浏览器内核`

##### 浏览器内核（渲染进程）

从前文我们得知，进程和线程是一对多的关系，也就是说一个进程包含了多条线程。

而对于`渲染进程`来说，它当然也是多线程的了，接下来我们来看一下渲染进程包含哪些线程。

- ```
  GUI渲染线程
  ```

  - 负责渲染页面，布局和绘制
  - 页面需要重绘和回流时，该线程就会执行
  - 与js引擎线程互斥，防止渲染结果不可预期

- ```
  JS引擎线程
  ```

  - 负责处理解析和执行javascript脚本程序
  - 只有一个JS引擎线程（单线程）
  - 与GUI渲染线程互斥，防止渲染结果不可预期

- ```
  事件触发线程
  ```

  - 用来控制事件循环（鼠标点击、setTimeout、ajax等）
  - 当事件满足触发条件时，将事件放入到JS引擎所在的执行队列中

- ```
  定时触发器线程
  ```

  - setInterval与setTimeout所在的线程
  - 定时任务并不是由JS引擎计时的，是由定时触发线程来计时的
  - 计时完毕后，通知事件触发线程

- ```
  异步http请求线程
  ```

  - 浏览器有一个单独的线程用于处理AJAX请求
  - 当请求完成时，若有回调函数，通知事件触发线程

##### 为什么 javascript 是单线程的

首先是历史原因，在创建 javascript 这门语言时，多进程多线程的架构并不流行，硬件支持并不好。

其次是因为多线程的复杂性，多线程操作需要加锁，编码的复杂性会增高。

而且，如果同时操作 DOM ，在多线程不加锁的情况下，最终会导致 DOM 渲染的结果不可预期。

##### 为什么 GUI 渲染线程与 JS 引擎线程互斥

这是由于 JS 是可以操作 DOM 的，如果同时修改元素属性并同时渲染界面(即 `JS线程`和`UI线程`同时运行)， 那么渲染线程前后获得的元素就可能不一致了。

因此，为了防止渲染出现不可预期的结果，浏览器设定 `GUI渲染线程`和`JS引擎线程`为互斥关系， 当`JS引擎线程`执行时`GUI渲染线程`会被挂起，GUI更新则会被保存在一个队列中等待`JS引擎线程`空闲时立即被执行。



