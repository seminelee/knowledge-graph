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

观察者模式

![ååç»å®](https://seminelee.github.io/static/2018/07/mvvm-1.png)

- Observer 监听者

  主要通过`Object`的`defineProperty`属性，重写data的`set`和`get`函数来实现的。每个属性的`set`函数中通知该属性的所有订阅者更新数据

- Compile 解析指令

  解析Vue指令`v-model`等，给对应的数据属性添加订阅者，绑定更新函数。初始化视图。

- Watcher 订阅者

  订阅者数组。Compile初始化视图后数据更新时，Observer通知变化，最后更新视图。
  
  *vue的更新dom的diff算法位置在订阅者更新自己的方法里面*



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



### 6.3.3 错误边界

如果一个 class 组件中定义了 [`static getDerivedStateFromError()`](https://zh-hans.reactjs.org/docs/react-component.html#static-getderivedstatefromerror) 或 [`componentDidCatch()`](https://zh-hans.reactjs.org/docs/react-component.html#componentdidcatch) 这两个生命周期方法中的任意一个（或两个）时，那么它就变成一个错误边界。当抛出错误后，请使用 `static getDerivedStateFromError()` 渲染备用 UI ，使用 `componentDidCatch()` 打印错误信息。

可以开发一个这样的高阶组件，它会捕获它的子组件的错误。

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

