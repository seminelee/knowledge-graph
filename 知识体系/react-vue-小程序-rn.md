# 6. React/Vue

## 6.1 虚拟dom

https://seminelee.github.io/2019/09/09/virtual-dom/

虚拟DOM简而言之就是，用JS去按照DOM结构来实现的树形结构对象，你也可以叫做DOM对象

实现思路（react）：

1. 用JS对象模拟DOM（虚拟DOM）
2. 把此虚拟DOM转成真实DOM并插入页面中（render）
3. 如果有事件发生修改了虚拟DOM，比较两棵虚拟DOM树的差异，得到差异对象（补丁数组）（diff）
4. 把差异对象（补丁数组）应用到真正的DOM树上（patch）

Vue在更新数据这一步骤实现的方式不相同，vue监听每个数据，更新数据时直接通知对应的订阅者，然后更新视图。这也使得Vue比起react可以更快地计算出Virtual DOM的差异，不需要重新渲染整个组件树。

### 6.1.1 diff算法

[浅析React Diff 与Fiber - 知乎](https://zhuanlan.zhihu.com/p/58863799)

传统 diff 算法 通过循环递归对节点进行依次对比。那React要怎么优化diff的过程呢

![img](https://pic1.zhimg.com/80/v2-0454c358dc97314557f3af224d110bf4_1440w.jpg)

1. Web UI 中 DOM 节点跨层级的移动操作特别少，可以忽略不计。

   ![img](https://pic2.zhimg.com/80/v2-91dc04a538ffb258e380328351153429_1440w.jpg)

2. 拥有相同类的两个组件将会生成相似的树形结构，拥有不同类的两个组件将会生成不同的树形结构。

   ![img](https://pic2.zhimg.com/80/v2-84a9a55b3f146c25cec76ddf82613a4d_1440w.jpg)

3. 对于同一层级的一组子节点，它们可以通过唯一 id 进行区分。

   ![img](https://pic1.zhimg.com/80/v2-ac6e272b0923f8fc9576aabe2c6f25c4_1440w.jpg)

基于以上三个前提策略，React 分别对 **tree diff**、**component diff** 以及 **element diff** 进行算法优化。

### 6.1.2 Fiber算法

众所周知，JavaScript在浏览器的主线程上运行，通常样式计算、布局以及页面绘制会一起运行。如果JavaScript运行时间过长，就会阻塞这些其他工作，可能导致掉帧。

而React在首次渲染过程中构建出Virtual DOM Tree，后续需要更新时（setState()），diff Virtual DOM Tree得到DOM change，并把DOM change应用（patch）到DOM树。

自顶向下的递归mount/update，无法中断（持续占用主线程），这样主线程上的布局、动画等周期性任务以及交互响应就无法立即得到处理，影响体验。同时也造成了React在一些响应体验要求较高的场景不适用，比如动画，布局和手势等。

Fiber解决这个问题的思路是增量更新。

> 把渲染/更新过程（递归diff）拆分成一系列小任务，每次检查树上的一小部分，做完看是否还有时间继续下一个任务，有的话继续，没有的话把自己挂起，主线程不忙的时候再继续。

## 6.2 Vue 双向绑定

https://seminelee.github.io/2018/07/21/vue-3/

![ååç»å®](https://seminelee.github.io/static/2018/07/mvvm-1.png)

- Observer 监听者

  主要通过`Object`的`defineProperty`属性，重写data的`set`和`get`函数来实现的。每个属性的`set`函数中通知该属性的所有订阅者更新数据

- Compile 解析指令

  解析Vue指令`v-model`等，给对应的数据属性添加订阅者，绑定更新函数。初始化视图。

- Watcher 订阅者

  订阅者数组。Compile初始化视图后数据更新时，Observer通知变化，最后更新视图。



## 6.3 react单向数据流

### 6.3.1 受控组件与非受控组件

__[受控组件](https://react.docschina.org/docs/forms.html)__

在 HTML 中，表单元素（如`input `）之类的表单元素通常自己维护 state，并根据用户输入进行更新。而在 React 中，可变状态（mutable state）通常保存在组件的 state 属性中，并且只能通过使用 `setState()`来更新。

我们可以把两者结合起来，使 React 的 state 成为“唯一数据源”。渲染表单的 React 组件还控制着用户输入过程中表单发生的操作。被 React 以这种方式控制取值的表单输入元素就叫做“受控组件”。

``` jsx
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};
    this.handleChange = this.handleChange.bind(this);
  }
  handleChange(event) {
    this.setState({value: event.target.value});
  }
  // ...
  render() {
    return (
      <input type="text" value={this.state.value} onChange={this.handleChange} />
    );
  }
}
```

__[非受控组件](https://react.docschina.org/docs/uncontrolled-components.html)__

要编写一个非受控组件，而不是为每个状态更新都编写数据处理函数，你可以 [使用 ref](https://react.docschina.org/docs/refs-and-the-dom.html) 来从 DOM 节点中获取表单数据。

因为非受控组件将真实数据储存在 DOM 节点中，所以在使用非受控组件时，有时候反而更容易同时集成 React 和非 React 代码。如果你不介意代码美观性，并且希望快速编写代码，使用非受控组件往往可以减少你的代码量。否则，你应该使用受控组件。

``` jsx
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.handleSubmit = this.handleSubmit.bind(this);
    this.input = React.createRef();
  }
  handleSubmit(event) {
    alert('A name was submitted: ' + this.input.current.value);
    event.preventDefault();
  }
  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" ref={this.input} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

##6.4 react-router与vue-router

以react-router为例，路由的实现主要有3种技术：

- 老浏览器的history: 主要通过hash来实现，对应`createHashHistory`。利用hash改变，网页不会刷新重载的特性。

  ``` js
  window.location.hash = path // push跳转
  window.location.replace(
      window.location.pathname + window.location.search + '#' + path
  ) // replace
  ```

- 高版本浏览器: 通过html5里面的history，对应`createBrowserHistory`

  ``` js
  window.history.pushState(historyState, null, path) // push跳转
  window.history.replaceState(historyState, null, path) // replace
  ```

- node环境下: 主要存储在内存memeory里面，对应`createMemoryHistory`

  ``` js
  entries.push(location) // push跳转
  entries[current] = location // replace
  ```

# 

## 6.5 对比

### 6.4.1 生命周期：

Vue

[https://cn.vuejs.org/v2/guide/instance.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%9B%BE%E7%A4%BA](https://cn.vuejs.org/v2/guide/instance.html)

![image-20190909232615607](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20190909232615607.png)

- beforeCreate-created：Observer监听属性
- created-beforeMount：compiler解析指令，绑定对应属性订阅者，生成dom tree
- beforeMount-mounted：虚拟dom转成真实dom
- mounted-beforeUpdate-updated：当有改动，Observer通知订阅者，生成新的dom tree，diff然后patch，更新视图
- beforeDestroy-destroy：解除订阅和子组件和event listeners

React

http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/

![image-20190909233833419](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20190909233833419.png)

- componentWillMount-componentDidMount：生成虚拟dom tree，转成真实的dom
- componentDidMount-componentWillUpdate：diff对比新旧dom tree
- componentWillUpdate-componentDidUpdate：patch，更新视图
- componentWillUnmount：卸载

### 6.4.2 相同点

- 都体现了虚拟dom思想
- 都鼓励组件化应用
- 生命周期也相似

### 6.4.3 区别

- Vue双向绑定；React单向数据流，大型应用状态更可控

- Vue模版指令，把html，css，js组合到一起，要记api；React all in js，jsx，api少

- react可以通过高阶组件（Higher Order Components--HOC）来扩展；vue需要通过mixins来扩展

- 对比新旧数据beforeupate的实现：Vue知道是组件数据中的value字段发生更新了， 而React只知道是组件的State发生了变化，并不知道是什么数据发生了变化。

  react对比新旧dom树，生成补丁数组，再更新视图；vue监听每个数据，更新数据时直接通知对应的订阅者，然后更新视图。

# 7. 小程序

## 7.1 小程序双线程模型

https://seminelee.github.io/2019/05/08/rn-miniprogram/

![å°ç¨åºé¡µé¢æ¸²æ](https://seminelee.github.io/static/2019/07/miniprogram-dom.png)

![å°ç¨åºåçº¿ç¨æ¨¡å](https://seminelee.github.io/static/2019/07/miniprogram.png)

小程序的运行环境分成渲染层和逻辑层，WXML 模板和 WXSS 样式工作在渲染层，JS 脚本工作在逻辑层。小程序的渲染层和逻辑层分别由2个线程管理：渲染层的界面使用了WebView 进行渲染；逻辑层采用JsCore线程运行JS脚本。
一个小程序存在多个界面，所以渲染层存在多个WebView线程。这使得小程序更贴近原生体验，也避免了单个WebView的任务过于繁重。

小程序的渲染层和逻辑层分离主要有两个原因：

1. UI渲染跟 JavaScript 的脚本执行分别在两个线程，从而避免一些逻辑任务抢占UI渲染的资源（详细看10.1.1浏览器渲染，浏览器中UI渲染线程与js线程互斥）。
2. 为了解决管控与安全问题，提供一个沙箱环境来运行开发者的JavaScript 代码（逻辑层），从而阻止开发者使用一些浏览器提供的，诸如跳转页面、操作DOM、动态执行脚本的开放性接口。
3. 渲染层和逻辑层的分离也给在不同的环境下（小程序与小程序开发者工具）运行提供了可能性。

### 7.1.1 与浏览器和nodejs中开发的区别

- 和浏览器的区别：
  1. 网页开发渲染线程和脚本线程是互斥的，这也是为什么长时间的脚本运行可能会导致页面失去响应，而在小程序中，二者是分开的，分别运行在不同的线程中（不互斥）。
  2. 小程序的逻辑层和渲染层是分开的，逻辑层运行在 JSCore 中，并没有一个完整浏览器对象，因而缺少相关的DOM API和BOM API。这一区别导致了前端开发非常熟悉的一些库，例如 jQuery、 Zepto 等，在小程序中是无法运行的。

- 和nodejs的区别：

  JSCore 的环境同 NodeJS 环境也是不尽相同，所以一些 NPM 的包在小程序中也是无法运行的。

  新版本支持npm包，但实际上只是放到miniprogram_npm文件夹里，require方法在导入模块时，也会从这个miniprogram_npm文件夹中查找模块

## 7.2 与hybrid app、React Native 对比

### 7.2.1 React Native

![React Native](https://seminelee.github.io/static/2019/07/react-native.jpeg)

### 7.2.2 共同点

- 都具有hybrid技术的优点，兼具“Native App良好用户交互体验的优势”和“Web App跨平台开发的优势”。
- 都实现了一套跨语言通讯方案，来完成 Native(Java/Objective-c/…)端 与 JavaScript （小程序中分为渲染层和逻辑层）的通讯。
- 小程序与react native都使用Web 相关技术来编写业务代码，都体现了虚拟dom的思想；

### 7.2.3 不同点

- hybrid app：webview渲染；热更新方便

- 小程序：大部分webview渲染，小部分由客户端参与渲染；热更新方便
- rn：客户端原生渲染；热更新比较复杂

> rn热更新：
>
> 服务器使用bsdiff算法将老RN包和新RN包生成一个补丁patch文件，供客户端下载。客户端下载patch文件，使用bspatch算法将补丁patch文件和老RN包生成一个新RN包。



## 7.3 小程序框架

小程序开发有哪些痛点?

- 频繁调用 setData及 setData过程中页面跳闪

- 组件化支持能力太弱(几乎没有)

- 不能使用 less、scss 等预编译器

### 7.3.1 wepy

wepy：vue+webpack

功能：脚手架、编译打包、核心库

编译原理：

1. index.wpy拆解成style、template、script，script中拆解出json文件
2. 编译前梳理组件引用关系与npm引用关系
3. 使用各种loader编译成wxss、wxml、js文件
4. 各种插件处理，代码混淆插件、wxml/图片压缩插件
5. 最终生成wxss、wxml、js、json

缺点：

1. 语法解析：编译`template`用的是xmldom，这个库已经没有人维护了等等，报错时无法很快定位是哪个地方报错了。
2. 类Vue语法，某些Vue的API不支持
3. 错误处理机制：报错没有详细的信息（位置等），比较难快速定位问题
4. 数据绑定性能优化：解决频繁`setData`的问题。diff方法的优化。

v2.0数据绑定优化： 

- v1.0
  初始化时对树进行深拷贝->当有修改时与拷贝的树相比较->得出脏数据->`setData`
- v2.0
  参考Vue并做了优化
  初始化时创建watcher->当有修改时watcher会记录修改（key-path-value），并将脏数据放到队列里->`setData` （不会做深比较了）

### 7.3.2 wepy和mpvue

语法规范不同：wepy使用类vue语法规范；mpvue使用vue语法规范

生命周期不同：wepy 生命周期基本与原生小程序相同；mpvue使用vue的生命周期

### 7.3.3 taro

使用 React.js 开发微信小程序的前端框架。同时因为使用了react的原因所以除了能编译h5, 小程序外还可以编译为ReactNative;



