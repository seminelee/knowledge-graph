[Flutter实战](https://book.flutterchina.club/)

# 移动开发技术简介

## 1 原生开发

原生应用程序是指某一个移动平台（比如iOS或安卓）所特有的应用，使用相应平台支持的开发工具和语言，并直接调用系统提供的SDK API。

优点：

- 可访问平台全部功能（GPS、摄像头）
- 速度快、性能高、可以实现复杂动画及绘制，整体用户体验好

缺点：

- 开发成本高；不同平台必须维护不同代码
- 内容固定，动态化弱，热更新比较麻烦，大多数情况下，有新功能更新时需要发版

随着移动互联网的发展，对应用动态化(不发版也可以更新应用内容)的需求迫在眉睫；维护Android、iOS两个开发团队，原生开发的开发成本和测试成本太高。

而时至今日，已经有很多跨平台框架(Android和iOS)，根据其原理，主要分为三类：

- H5+原生混合开发（Cordova、Ionic、微信小程序）
- JavaScript开发+原生渲染 （React Native、Weex、快应用）
- 自绘UI+原生(QT for mobile、Flutter)



## 2 H5+原生混合开发（Hybrid技术）

即H5+原生混合开发，这类框架主要原理就是将APP的一部分需要动态变动的内容通过H5来实现，通过原生的网页加载控件WebView (Android)或WKWebView（iOS）来加载（以后若无特殊说明，我们用WebView来统一指代android和iOS中的网页加载控件）。

如之前所述，原生开发可以访问平台所有功能，而混合开发中，H5代码是运行在WebView中，而WebView实质上就是一个浏览器内核，其JavaScript依然运行在一个权限受限的沙箱中，所以对于大多数系统能力都没有访问权限，如无法访问文件系统、不能使用蓝牙等。

而混合框架一般都会在原生代码中预先实现一些访问系统能力的API， 然后暴露给WebView以供JavaScript调用，这样一来，WebView就成为了JavaScript与原生API之间通信的桥梁，主要负责JavaScript与原生之间传递调用消息，而消息的传递必须遵守一个标准的协议，它规定了消息的格式与含义，我们把依赖于WebView的用于在JavaScript与原生之间通信并实现了某种消息传输协议的工具称之为**WebView JavaScript Bridge**, 简称 **JsBridge**，它也是混合开发框架的核心。

- 优点

  动态内容是H5，web技术栈，社区及资源丰富；热更新容易，开发成本较低

- 缺点

  性能不好，对于复杂用户界面或动画，WebView不堪重任。

### 2.1 微信小程序双线程模型

混合框架一般是web渲染。

微信小程序则是使用浏览器内核来渲染界面，小部分原生组件由客户端参与渲染。即界面主要由成熟的 Web 技术渲染，辅之以大量的接口提供丰富的客户端原生能力。

微信小程序的另一个特色是：小程序的[渲染层和逻辑层]([https://developers.weixin.qq.com/miniprogram/dev/framework/quickstart/framework.html#%E6%B8%B2%E6%9F%93%E5%B1%82%E5%92%8C%E9%80%BB%E8%BE%91%E5%B1%82](https://developers.weixin.qq.com/miniprogram/dev/framework/quickstart/framework.html#渲染层和逻辑层))分别由2个线程管理，渲染层的界面使用了WebView（一个页面一个webview） 进行渲染；逻辑层采用JsCore线程运行JS脚本。视图层与逻辑层间的通信和逻辑层的网络请求经由Native转发。



## 3 JavaScript开发+原生渲染

React Native、Weex等

React Native使用js语言，类似于html的jsx，以及CSS来开发移动应用。RN和React原理相同，Flutter也是受React启发。

### 3.1React：DOM树和响应式编程

React的两个重要概念：DOM树和响应式编程

- DOM树

  DOM树通常指HTML对应的渲染树，也可以指Android中的控件树。前者常用于Web开发中，后者常用于原生开发中。

- 响应式编程

  React中提出一个重要思想：状态改变则UI随之自动改变，React框架就是响应用户状态改变的事件而执行重新构建用户界面的工作，这就是响应式编程。

  开发者只需关注状态转移（数据），当状态发生变化时，React框架会通过Diff算法，根据当前渲染树计算出树中变化的部分，然后更新变化的部分（DOM操作），重新构建UI。

  注意，这里并不会直接计算和渲染真实DOM树的变化部分，而是先建立一个抽象层，即 __虚拟DOM树__，改动先高效地同步到虚拟DOM，最后再 __批量__ 同步到真实DOM中。这是因为在浏览器中每一次DOM操作都有可能引起浏览器的重绘或回流，如果每次改变都直接对DOM进行操作，这回带来性能问题。而批量操作只会触发一次DOM更新。

  > 重绘：DOM只是外观风格发生变化（如颜色）
  >
  > 回流：DOM树的结构发生变化（如尺寸、布局、节点隐藏等导致）

### 3.2 React Native

React Native与React的区别在于虚拟DOM映射的对象。React中虚拟DOM最终映射为浏览器DOM树；React Native中虚拟DOM会通过 __JavaScriptCore__映射为原生控件树。这个过程分两步：

1. 将虚拟DOM布局信息传递给原生
2. 原生根据布局信息通过对应控件渲染控件树

__JavaScriptCore__是一个JavaScript解释器，它在React Native中主要有两个作用：

1. 为JavaScript提供运行环境
2. 是JavaScript与原生app之间通信的桥梁，作用和JsBridge一样。在iOS中，很多JsBridge的实现都基于JavaScriptCore。

- 优点：
  - RN实现了跨平台（但有部分api两个平台之间有区别）
  - 原生控件渲染，性能比hybrid app中的H5好
  - 使用前端开发熟悉的web开发技术栈，社区庞大、上手快、开发成本相对较低。
- 缺点
  - 渲染时需要JavaScript和原生之间通信，在有些场景如拖动可能会因为通信频繁导致卡顿。
  
    > 因为在滑动和拖动过程往往都会引起布局发生变化，所以JavaScript需要和Native之间不停的同步布局信息，这和在浏览器中要JavaScript频繁操作DOM所带来的问题是相同的，都会带来比较可观的性能开销。
  
  - JavaScript为脚本语言，执行时需要JIT(Just In Time，边编译边执行)，执行效率和AOT(Ahead Of Time，编译后再执行)代码仍有差距。
  
  - 由于渲染依赖原生控件，不同平台的控件需要单独维护，并且当系统更新时，社区控件可能会滞后；除此之外，其控件系统也会受到原生UI系统限制，例如，在Android中，手势冲突消歧规则是固定的，这在使用不同人写的控件嵌套时，手势冲突问题将会变得非常棘手。

> __Weex__
>
> 思想与原理与RN类似，最大的不同是语法层面。Weex支持Vue语法和Rax语法（基于React JSX语法）。而RN只支持JSX语法。
>
> __快应用__
>
> 快应用是华为、小米、OPPO、魅族等国内9大主流手机厂商共同制定的轻量级应用标准，目标直指微信小程序。它也是采用JavaScript语言开发，原生控件渲染，与React Native和Weex相比主要有两点不同：
>
> 1. 快应用自身不支持Vue或React语法，其采用原生JavaScript开发，其开发框架和微信小程序很像，值得一提的是小程序目前已经可以使用Vue语法开发（mpvue），从原理上来讲，Vue的语法也可以移植到快应用上。
> 2. React Native和Weex的渲染/排版引擎是集成到框架中的，每一个APP都需要打包一份，安装包体积较大；而快应用渲染/排版引擎是集成到ROM中的，应用中无需打包，安装包体积小，正因如此，快应用才能在保证性能的同时做到快速分发。



## 4 自绘UI+原生

这种技术的思路是，通过在不同平台实现一个统一接口的渲染引擎来绘制UI，而不依赖系统原生控件，从而做到不同平台UI的一致性。

### 4.1 QT Mobile

Qt是一个1991年由Qt Company开发的跨平台C++图形用户界面应用程序开发框架。

但由于移动开发社区太小，生态不好；官方推广不利；C++开发比起Web开发栈的劣势等在移动端表现不佳。

优点：

- 性能高。自绘UI引擎是直接调用系统API来绘制UI，所以性能和原生控件接近。在布局过程中不需要像RN那样要在JavaScript和Native之间通信。
- 灵活、组件库易维护、UI外观保真度和一致性高；代码容易维护，不需要根据不同平台的控件单独维护。

缺点：

- 动态性不足，热重载较麻烦。为了保证UI绘制性能，自绘UI系统一般都会采用AOT模式编译其发布包，不能像Hybrid和RN那些使用JavaScript（JIT）作为开发语言的框架那样动态下发代码。
- 开发效率较低：如QT使用的C++，作为静态语言，在UI开发方面灵活性不及JavaScript这样的动态语言，且没有JavaScript和Java中的垃圾回收（GC）机制。

### 4.2 Flutter

Flutter是Google发布的一个用于创建跨平台、高性能移动应用的框架。Flutter和QT mobile一样，都没有使用原生控件，相反都实现了一个自绘引擎，使用自身的布局、绘制系统。

Flutter使用自己的高性能渲染引擎来绘制widget，使用Skia作为其2D渲染引擎。

> Skia是Google的一个2D图形处理函数库，包含字型、坐标转换，以及点阵图都有高效能且简洁的表现，Skia是跨平台的，并提供了非常友好的API，目前Google Chrome浏览器和Android均采用Skia作为其绘图引擎。

Flutter APP采用Dart语言开发。

- **基于JIT的快速开发周期**：Flutter在开发阶段采用，采用JIT模式，这样就避免了每次改动都要进行编译，极大的节省了开发时间；

- **基于AOT的发布包**: Flutter在发布时可以通过AOT生成高效的ARM代码以保证应用性能。而JavaScript则不具有这个能力。
- Dart是类型安全的语言，支持静态类型检测

比起QT Mobile有以下优势：

- 生态好，社区庞大，文档齐全
- 技术支持强，Google正在大力推进
- 开发效率高：Flutter的热重载可帮助开发者快速地进行测试、构建UI、添加功能并更快地修复错误。在iOS和Android模拟器或真机上可以实现毫秒级热重载，并且不会丢失状态。



## 5 总结

| 技术类型            | UI渲染方式      | 性能 | 开发效率        | 动态化     | 框架代表       |
| ------------------- | --------------- | ---- | --------------- | ---------- | -------------- |
| H5+原生             | WebView渲染     | 一般 | 高              | 支持       | Cordova、Ionic |
| JavaScript+原生渲染 | 原生控件渲染    | 好   | 中              | 支持       | RN、Weex       |
| 自绘UI+原生         | 调用系统API渲染 | 好   | Flutter高, QT低 | 默认不支持 | QT、Flutter    |



- 开发语言主要指UI的开发语言。

- 开发效率，是指整个开发周期的效率，包括编码时间、调试时间、以及排错、兼容时间。

- 动态化主要指是否支持动态下发代码和是否支持热更新。

  值得注意的是Flutter的Release包默认是使用Dart AOT模式编译的，所以不支持动态化，但Dart还有JIT或snapshot运行方式，这些模式都是支持动态化的。



# 小程序

## 1 小程序双线程模型

https://seminelee.github.io/2019/05/08/rn-miniprogram/

![å°ç¨åºé¡µé¢æ¸²æ](https://seminelee.github.io/static/2019/07/miniprogram-dom.png)

 ![img](https://user-gold-cdn.xitu.io/2018/5/17/1636cb90ba54f91e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 

- **逻辑层：创建一个单独的线程去执行 JavaScript，在这里执行的都是有关小程序业务逻辑的代码，负责逻辑处理、数据请求、接口调用等**
- **视图层：界面渲染相关的任务全都在 WebView 线程里执行，通过逻辑层代码去控制渲染哪些界面。一个小程序存在多个界面，所以视图层存在多个 WebView 线程**
- **JSBridge 起到架起上层开发与Native（系统层）的桥梁，使得小程序可通过API使用原生的功能，且部分组件为原生组件实现，从而有良好体验**

小程序的渲染层和逻辑层分离主要有两个原因：

1. UI渲染跟 JavaScript 的脚本执行分别在两个线程，从而避免一些逻辑任务抢占UI渲染的资源（详细看10.1.1浏览器渲染，浏览器中UI渲染线程与js线程互斥）。
2. 为了解决管控与安全问题，提供一个沙箱环境来运行开发者的JavaScript 代码（逻辑层），从而阻止开发者使用一些浏览器提供的，诸如跳转页面、操作DOM、动态执行脚本的开放性接口。
3. 渲染层和逻辑层的分离也给在不同的环境下（小程序与小程序开发者工具）运行提供了可能性。

### 1.1 与浏览器和nodejs中开发的区别

- 和浏览器的区别：

  1. 网页开发渲染线程和脚本线程是互斥的，这也是为什么长时间的脚本运行可能会导致页面失去响应，而在小程序中，二者是分开的，分别运行在不同的线程中（不互斥）。
  2. 小程序的逻辑层和渲染层是分开的，逻辑层运行在 JSCore 中，并没有一个完整浏览器对象，因而缺少相关的DOM API和BOM API。这一区别导致了前端开发非常熟悉的一些库，例如 jQuery、 Zepto 等，在小程序中是无法运行的。

- 和nodejs的区别：

  JSCore 的环境同 NodeJS 环境也是不尽相同，所以一些 NPM 的包在小程序中也是无法运行的。

  新版本支持npm包，但实际上只是放到miniprogram_npm文件夹里，require方法在导入模块时，也会从这个miniprogram_npm文件夹中查找模块

## 2 与hybrid app、React Native 对比

### 2.1 React Native

![React Native](https://seminelee.github.io/static/2019/07/react-native.jpeg)

### 2.2 共同点

- 都具有hybrid技术的优点，兼具“Native App良好用户交互体验的优势”和“Web App跨平台开发的优势”。
- 都实现了一套跨语言通讯方案，来完成 Native(Java/Objective-c/…)端 与 JavaScript （小程序中分为渲染层和逻辑层）的通讯。
- 小程序与react native都使用Web 相关技术来编写业务代码，都体现了虚拟dom的思想；

### 2.3 不同点

- hybrid app：webview渲染；热更新方便

- 小程序：大部分webview渲染，小部分由客户端参与渲染；热更新方便
- rn：客户端原生渲染；热更新比较复杂

> rn热更新：
>
> 服务器使用bsdiff算法将老RN包和新RN包生成一个补丁patch文件，供客户端下载。客户端下载patch文件，使用bspatch算法将补丁patch文件和老RN包生成一个新RN包。



## 3 小程序框架

小程序开发有哪些痛点?

- 频繁调用 setData及 setData过程中页面跳闪

- 组件化支持能力太弱(几乎没有)

- 不能使用 less、scss 等预编译器

### 3.2 wepy和mpvue

语法规范不同：wepy使用类vue语法规范；mpvue使用vue语法规范

生命周期不同：wepy 生命周期基本与原生小程序相同；mpvue使用vue的生命周期

### 3.3 taro

使用 React.js 开发微信小程序的前端框架。同时因为使用了react的原因所以除了能编译h5, 小程序外还可以编译为ReactNative;

小程序插件开发有bug



## 4 优化措施

- 避免频繁的setData、避免每次setData传递大量的新数据。

   由`setData`的底层实现可知，我们的数据传输实际是一次 `evaluateJavascript` 脚本过程，当数据量过大时会增加脚本的编译执行时间，占用 WebView JS 线程， 

-  注意大图片和长列表图片的使用。

  在 iOS 上，小程序的页面是由多个 WKWebView 组成的，在系统内存紧张时，会回收掉一部分 WKWebView。从过去我们分析的案例来看，大图片和长列表图片的使用会引起 WKWebView 的回收。 

- 分包加载， 优化小程序首次启动的下载时间 ； **清理没有使用到的代码和资源** ，减少代码包大小

- 部分页面可以预先加载数据