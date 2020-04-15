# 5. nodejs

## 5.1 nodejs的架构与事件循环

https://seminelee.github.io/2019/01/26/event-loop/

Node.js 最大的特点就是使用 **异步式 I/O** 与 **事件驱动** 的架构设计

Node.js的运行机制如下:

- V8 引擎解析 JavaScript 脚本。
- 解析后的代码，调用 Node API。
- libuv 库负责 Node API 的执行。它将不同的任务分配给不同的线程，形成一个 Event Loop（事件循环），以异步的方式将任务的执行结果返回给 V8 引擎。
- V8 引擎再将结果返回给用户。

event loop即事件循环，是指浏览器或Node的一种解决javaScript单线程运行时不会阻塞的一种机制，也就是我们经常使用异步的原理。

6个阶段

- timers: 执行setTimeout和setInterval中到期的callback。
- pending callback: 上一轮循环中少数的callback会放在这一阶段执行。
- idle, prepare: 仅在内部使用。
- poll: 最重要的阶段，执行pending callback，在适当的情况下会阻塞在这个阶段。
- check: 执行setImmediate的callback。
- close callbacks: 执行close事件的callback，例如socket.on(‘close’[,fn])或者http.server.on(‘close, fn)。

event loop的每一次循环都需要依次经过上述的阶段。每个阶段都有自己的FIFO的callback队列（在timer阶段其实使用一个最小堆而不是队列来保存所有元素，比如timeout的callback是按照超时时间的顺序来调用的，并不是先进先出的队列逻辑），每当进入某个阶段，都会从所属的队列中取出callback来执行。当队列为空或者被执行callback的数量达到系统的最大数量时，进入下一阶段。这六个阶段都执行完毕称为一轮循环。

## 5.2 nodejs与php对比 与性能优化

https://seminelee.github.io/2017/11/19/php/

### 5.2.1 与php对比

- php多进程，Node.js是单进程

  如果PHP代码损坏，只影响一个进程，不会拖垮整个服务器；而在Node.js环境中，所有的请求均在单一的进程服务中，当某个请求导致未知错误时，整个服务器都会受到影响。

- php多线程，Node.js单线程，高并发的处理方式不同

  Node.js事件循环机制，将不同的异步任务分配给不同的线程，回调函数放到任务队列中，主线程按一定的规则执行，解决javaScript单线程运行时不会阻塞，速度快但占内存

  php借用Apache服务器提供多线程服务

> 进程在运行过程中能够申请创建和使用系统资源（如独立的内存区域等），这些资源也会随着进程的终止而被销毁。
>
> 而线程则是进程内的一个独立执行单元，没有自己的地址空间，与进程内的其他线程一起共享分配给该进程的所有资源。
> 区别：
> （1）进程具有独立的空间地址，一个进程崩溃后，在保护模式下不会对其它进程产生影响。
> （2）线程只是一个进程的不同执行路径，线程有自己的堆栈和局部变量，但线程之间没有单独的地址空间，一个线程死掉就等于整个进程死掉。

 - 都是弱类型

### 5.2.2 性能优化

- 使用最新版本的 Node.js（V8引擎等升级）

- 使用 [fast-json-stringify](https://github.com/fastify/fast-json-stringify) 加速 JSON 序列化

- `Promise.all()` 的并行能力（多个没有连续关系的异步请求）

- 对于大文件，我们不需要把它完全读入内存`fs.readFile`，而是使用 Stream 流式地把它发送出去`fs.createReadStream`

  > `createReadStream`是给你一个ReadableStream，你可以听它的`data`，一点一点儿处理文件，用过的部分会被GC（垃圾回收），所以占内存少。 readFile是把整个文件全部读到内存里。

- 使用 node-clinic 快速定位性能问题

- 不要让静态资源使用Node.js cdn是更好的选择

- 减少请求量 返回体体积



## 5.3 koa

[Egg.js 与 Koa]( https://eggjs.org/zh-cn/intro/egg-and-koa.html )

 [Koa](https://koajs.com/) 是一个新的 web 框架，由 Express 幕后的原班人马打造。 Koa 和 Express 的设计风格非常类似，底层也都是共用的[同一套 HTTP 基础库](https://github.com/jshttp)，  但是有几个显著的区别 ：

### 5.3.1  默认异步解决方案 

基于ES7开发 `async/await`

 通过同步方式编写异步代码带来的另外一个非常大的好处就是异常处理非常自然，使用 `try catch` 就可以将按照规范编写的代码中的所有错误都捕获到。 

### 5.3.2 中间件

下面的“洋葱”图更能形象地表达。用户的请求先后经过登陆管理，错误处理，各种中间件，应用。响应逆向经过这些步骤传递出去。

 ![img](https://camo.githubusercontent.com/d80cf3b511ef4898bcde9a464de491fa15a50d06/68747470733a2f2f7261772e6769746875622e636f6d2f66656e676d6b322f6b6f612d67756964652f6d61737465722f6f6e696f6e2e706e67) 



### 5.3.3 Context

  Context 上也挂载了 Request 和 Response 两个对象。和 Express 类似，这两个对象都提供了大量的便捷方法辅助开发。

## 5.4 egg

 Egg 选择了 Koa 作为其基础框架，在它的模型基础上，进一步对它进行了一些增强。 

### 5.4.1 扩展

 我们可以通过定义 `app/extend/{application,context,request,response}.js` 来扩展 Koa 中对应的四个对象的原型，通过这个功能，我们可以快速的增加更多的辅助方法，例如我们在 `app/extend/context.js` 中写入下列代码： 

``` js
// app/extend/context.js
module.exports = {
  get isIOS() {
    const iosReg = /iphone|ipad|ipod/i;
    return iosReg.test(this.get('user-agent'));
  },
};
```

 在 Controller 中，我们就可以使用到刚才定义的这个便捷属性了： 

``` js
// app/controller/home.js
exports.handler = ctx => {
  ctx.body = ctx.isIOS
    ? 'Your operating system is iOS.'
    : 'Your operating system is not iOS.';
};
```



## 5.5 SQL 与 NoSQL

NoSQL(NoSQL = Not Only SQL )，指的是非关系型的数据库。NoSQL有时也称作Not Only SQL的缩写，是对不同于传统的关系型数据库的数据库管理系统的统称。

大多数都是分布式系统（distributed system）由多台计算机和通信的软件组件通过计算机网络连接（本地网络或广域网）组成。

优点：

- 可靠性：一台服务器的系统崩溃并不影响到其余的服务器。
- 可扩展性与灵活

- 更快：分布式计算系统可以有多台计算机的计算能力，使得它比其他系统有更快的处理速度
- 更好的性价比

缺点：

- 故障排除较难
- 没有标准化
- 有限的查询功能（到目前为止）（相比于SQL的join等功能）

各种类型：

- key-value存储：DCache
- 图存储：Neo4j
- 对象存储等
