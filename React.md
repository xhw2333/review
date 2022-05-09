主要见笔记

# fiber

> 说一下fiber吧

可以说是一种纯js对象，一个执行单元。每执行一个执行单元，react会检查现在还剩多少时间，没有时间则把控制权让出去。

关键特性：

1. 增量渲染，即将任务拆分，均匀到每一帧去执行
2. 暂停、终止、复用渲染任务
3. 不同 更新优先级
4. 并发方面新的基础能力

流程：

![image-20220327111912814](C:\Users\小浩王\AppData\Roaming\Typora\typora-user-images\react——fiber.png)

requestIdleCallback：在浏览器空闲时执行此方法

react自己模拟实现了requestIdleCallback这个API，大概原理就是：通过判断有无剩余时间，来执行fiber单元（预留5ms去执行任务）



# diff算法

聊到diff算法，先说一下虚拟DOM

## 虚拟DOM

我的理解：虚拟DOM的出现就是为了减少频繁去操纵真实DOM，减少回流、重绘等操作，减少一些不当的操作（如下面的例子）

eg：像那种innerHTML大量数据的操作，用虚拟DOM的话就不需要执行多次innerHTML操作，只需一次

不然你每次去操作真实DOM，都会触发回流/重绘操作，给浏览器造成一定的性能损耗。

虚拟DOM的好处：可维护性强和减少心智负担



react的diff算法基于以下策略来制定

- 只对于同级节点进行比较，同级节点指的就是同个父亲节点的节点
- 不同类型的组件会创建新的不同树形结构
- 对于同层级的节点，需通过==key==来确定节点需要做出什么变化，保持稳定的状态



## tree diff

只对于同级节点进行比较，即使出现跨层级的树形结构，比如一个节点以及其子孙在新的要渲染的树的另一层，也不会渲染复用之前的旧节点，会重新创建这个和原来节点一模一样的节点。

## component diff

对于不同类型的组件，比如说一个类型为div的组件，新组件为一个类型为p的组件，即使他们同级，子孙节点的结构也几乎一样，但也不会复用原来的节点，会重新创建这个类型为p的组件，然后再创建其子孙节点。

## element diff

这里主要说**element diff**，主要有三种操作：**插入节点、删除节点、移动节点**

遵循**递增**原则，就是说从旧节点序列中按递增顺序找到从新节点序列开始的可以复用的节点，在此基础上进行对其添加、移动、删除节点操作。

这里举个例子说明，比较容易理解

旧：A->B->C->D （fiber树、链表结构）

新：B->C->E->A（ReactElement元素，数组结构）

说一下过程：（省略标记的步骤，以后详细说）

1. 首先会从新节点序列（数组）开始，这里是B，找到其在旧节点序列（链表）中的位置，位置为1，到时可以复用B，记录此时位置lastIndex=1
2. 找C，因为位置2>lastIndex，所以复用C，更新lastIndex为2，
3. 找E，没有则说明这个为新节点，到时需要**插入**
4. 找A，位置为0小于lastIndex，说明到时需要进行**移动**
5. 最后 ，对旧节点序列进行处理，复用B、C，在此添加E，移动A，删除D。



## 当新fiber节点和旧fiber节点key、type相同时，但props不同时，diff又如何操作？

​	基于**旧fiber节点和新fiber节点的props**去克隆出一个**fiber节点**，就是所谓fiber节点复用。



**整棵树都会进行diff算法**，但是会分情况看是否需要diff：

1. reconcile：将fiber和jsx比较，即我们通常说的diff算法

2. bailout：不需要reconcile，直接使用旧fiber节点

   进入bailout的条件：

   1. oldProps === newProps
   2. context没有变化
   3. type相等，组件类型相等
   4. fiber的lanes无任务，即没有更新任务

   

   **bailout又分两个级别：**

   - 本身不需要reconcile

   - 本身及子树不要reconcile

     > 通过**fiber.childLane**来判断

   

​		

# 如何优化首屏渲染

- 请求时
  - 看宿主环境：webview，即端内，进行预加载、预解析
  - webview：先来个预热

- 运行时
  - SSR
  - SSG



# 性能优化

react做的性能优化主要有以下两点

1. eagerState，不触发更新就叫做`eagerState`，比如一直`update(1)`
2. bailout。useMemo、shouldCompoentUpdate、React.memo是为了命中bailout



# react生命周期

## 旧版生命周期

首先说一下旧版的生命周期，分为三个阶段：初始化阶段、更新阶段、卸载阶段

### 初始化、挂载阶段

组件初次在DOM树中被渲染的过程

由`ReactDOM.render()`触发，首次渲染

1. constructor
2. componentWillMount
3. render
4. componentDidMount

### 更新阶段

组件状态发生变化，重新渲染的过程

由**`组件内部this.setState`或者父组件render**时触发，组件要进行更新

1. componentWillReceiveProps
2. componentWillUpdate
3. shouldComponentUpdate
4. render
5. componentDidUpdate

### 卸载阶段

组件被移除DOM树的过程

由`ReactDOM.unmountComponentAtNode()`触发

1. componentWillUnmount



## 新版生命周期

分为三个阶段，挂载、更新、卸载

### 挂载

1. constrcutor
   - 状态初始化
   - 给事件处理绑定this

2. getDerivedStateFromProps（nextProps，preState）
   - 静态方法，里面不能调用this

3. render
4. componentDidMount
   - 执行依赖DOM的操作
   - 发送网络请求
   - 添加订阅消息




### 更新

设置新state，forUpdate，props

1. getDerivedStateFromProps
2. shouldComponentUpdate
3. render
4. getSnapShotBeforeUpdate（preProps，preState）
   - 要和componentDidupdate一起使用
   - 返回的参数（默认null）作为componentDidUpdate的第三个参数
   - 在最终决定的render之前执行，保证获取到的状态与didUpdate中获取到的状态相同。

5. componentDidUpdate（preProps，preState，snapshot）



### 卸载

1. componentWillUnmount
   - 取消事件订阅




# 组件间通信

## 父子传值

- 父传子：通过props，直接传即可
- 子传父：父组件通过props传给子组件可以修改状态的函数，到时即可在子传父

## 祖孙、兄弟传值

**核心：**利用**context**

调用`React.createContext`，返回一个context对象，使用里面的**Provider、Consumer组件**，通过设置Provider组件的value，可以在Consumer组件里通过回调函数获取到这个value

```js
import React, { Component } from "react";

const { Provider, Consumer } = React.createContext();

export default class GPS extends Component {
  state = {
    info: "来自GPS的值",
  };

  render() {
    return (
      <Provider value={this.state.info}>
        GPS组件
        <Parent />
      </Provider>
    );
  }
}

class Parent extends Component {
  state = {};

  render() {
    return (
      <div>
        Parent组件
        <Son />
      </div>
    );
  }
}

class Son extends Component {
  render() {
    return (
      <Consumer>
        {/* Son组件 */}
        {(info) => {
          return <div>{info}</div>;
        }}
      </Consumer>
    );
  }
}
```

## 第三方工具

- pub-sub方法



# 函数组件和类式组件的区别

类式组件需要实例化，函数组件直接调用即可，所以函数组件性能更好

|                | 函数组件 | 类式组件 |
| -------------- | -------- | -------- |
| 是否有this     | 没有     | 有       |
| 是否有生命周期 | 没有     | 有       |
| 是否有state    | 没有     | 有       |



# 声明组件的方式

- function(){} //无状态函数组件
- React.Component //类式组件
- React.createClass() //es6之前



# 受控组件和非受控组件

受控组件：受react控制的组件，即需要为每个状态去编写一个事件处理程序，更新组件的state

非受控组件：不受react控制的组件，即不需编写事件处理程序去更新state，而是通过ref去操作真实DOM元素，获取对应的值



# ref的作用

ref可以操作react组件实例和DOM元素，不能render里使用ref，因为真实DOM还没有挂载到界面

```js
render(){
    getSon(){
        console.log(this.son);
    }
      
    return (
    	<div>
        	<Son ref={c=>this.son=c}/>
			<button onClick={this.getSon}>【btn】<button/>
        </div>
    )
}
```



# 合成事件

react利用**事件委托机制**，在**document**中去处理事件，我们写的事件没绑定到真实的DOM上，而是通过事件冒泡到document上，然后再去进行处理。

> 冒泡到document上的也不是原生事件，而是react实现的合成事件，所以要阻止冒泡应该采用`event.preventdefault`，而不是`event.stopProppagation`
>
> react17.0将事件处理添加到==渲染react树的根容器中==

**目的**：

- 抹平浏览器之间的兼容问题，跨平台
- 减少内存消耗，如果在每个DOM绑定事件，会消耗更多的内存。利用事件委托 ，可以更好管理事件的创建的销毁

**具体流程（简单说一下）**

- 监听原生`listener`：将fiber和DOM联系，将原生事件派发到react体系中
- 收集`listeners`：遍历fiber树，收集监听本事件的所有`listener`函数
- 派发合成事件：构建合成事件，将listeners加入派发队列，之后再对派发队列进行操作，取出`listeners`，遍历`listeners`进行派发 



事件合成-->事件绑定-->事件触发



# React.PureComponent

PureComponent表示一个纯组件，会减少render函数的调用，提高组件性能

PureComponent会自动执行shouldComponentUpdate这个生命周期函数，通过比较state和props是否有变化，来返回true或者false，减少组件的重新渲染，提高性能

> 只是作了一层**浅比较**

# React.memo

接收两个参数，第一个参数为组件，第二个类型为函数，可以做比对，返回true或者false来决定此组件是否需要重新渲染。

```jsx
React.memo(function(){
    return (
        <div>你好</div>
    )
},(props,nextProps)=>{
    return props === nextProps
})
```

> 只是作了一层**浅比较**



# React高阶组件、Render props、Hook

解决**代码复用**问题的主要方式

- React高阶组件（HOC），HOC是一个函数，参数为组件，返回一个新组件

  - 缺点：
    - 相同命名的props会覆盖老属性
    - 多层嵌套，冗杂耦合

  例子：

  可以用来渲染劫持

  ```jsx
  function HOC(WrappedComponent){
      return class Test extends React.Component{
          constructor(props){
  			super(props);
              state = {
                  //进行一些处理
                  data: handle()
              }
          }
          
          render(){
              return (
                  <WrappedComponent data={this.state.data} {...this.props}></WrappedComponent>
              )
          }
      }
  }
  
  class T extends React.Component{
      render(){
          
          <>Hello</>
      }
  }
  
  const Test = HOC(T);
  ```

  

- render prop：在react组件之间采用值为函数的prop共享代码

  ```jsx
  class index2 extends Component {
    render() {
      return (
        <div>
            {this.props.render({name:'xhw'})}
        </div>
      )
    }
  }
  
  //调用
  <Index2 render={(data)=>(
  	<h1>你的名字：{data.name}</h1>
  )}/>
  ```

- Hook：16.8新增的特性

  - 解决HOC的props重名问题
  - 解决render prop回调地狱问题



# React插槽（portals）

将组件可脱离父组件层级挂载在DOM树的任何位置

```jsx
ReactDOM.createPortal(child,container);
```

- 第一个参数child是可渲染的react子项
- 第二个参数container是一个**DOM元素**



# props.children

父组件在子组件传递值，然后在子组件里调`props.children`呈现出来

```jsx
function Parent(props){
    return (
 		<div>
        	<Child>
                <div>来自父组件</div>
            </Child>
        </div>   	
    )
}

function Children(props){
	return (
    	<>
        	{props.children}
        </>
    )
}
```



# 关于setState

## 流程

- 调用`setState`入口函数，相当于一个分发器，根据参数不同（区别有没有**回调函数**）将其分发到不同功能函数中
- 接着调用`enqueueSetState`将新的state放进状态队列
- 调用`enqueueUpdate`去实现组件实例的更新
  - 根据`isBatchingUpdates`判断是否有组件更新在执行，进行操作
  - 有则排队等待，
  - 无则进行更新

- 最后通过调用`schedulerUpdateOnFiber`方法去进行更新（凡是涉及到渲染或更新都会调用此api）



## setState是异步还是同步

设想如果每次调用setState，页面就更新一次，性能就十分损耗，不太现实，所以react做了处理

- 如果react是控制的了，比如合成事件、生命周期等等这些地方，那setState就是**异步**的，这样你获取到的state是之前设置的值
- 如果react控制不了，比如在原生事件、setTimeout等等这些地方，那setState就是**同步**的，这样你获取到的state就是现在更新后的值

> 当然以上是legacy模式下，如果在concurrent模式下，那就是不一样的情况，setState就是异步，毕竟涉及到scheduler调度中心。



## setState的批量更新（批处理）

调用setState，不会立即更新，会存入一个队列，react会在合适的时机，将多个状态修改合并成一个状态修改。



# React SSR 服务端渲染

服务端渲染是数据和模板组成的HTML，即HTML = 数据 + 模板，组件和页面在服务端生成HTML字符串，然后发送给浏览器，最后将静态标记“混合”为客户端可以交互的应用程序

特点：

- 利于SEO
- 模板、图片等资源存储在服务端



# React源码

只要涉及到改变fiber的操作（首次挂载或后续渲染），都会间调用`schedulerUpdateOnFiber`函数

此函数的逻辑主要是：

1. 直接进行fiber树构造，不进行调度，即直接`performSyncWork`
2. 注册调度任务，进行scheduler包的调度，间接进行fiber树构造。即`ensureRootIsScheduled`,这个函数会涉及到scheduler与react的链接。

## 优先级

主要涉及三种优先级

- react事件优先级
  - 点击事件、输入事件之类，优先级最高
- lane优先级
  - 通常setState会产生update对象，update里就有一个lane属性
  - 车道模型，32位表示，最高位符号位，低位（1越靠右）表示优先级越高
- scheduler优先级

# Scheduler调度中心

通过scheduledCallback（`unstable_scheduledCallback`）注册调度任务，接收参数：优先级、`callback`（其实就是fiber树构造任务`performConcurrentWorkOnRoot`）、`options`。

其中分为**过期任务以及非过期任务**：根据`expirationTime`区分

1. timer：非过期任务
2. task：过期任务

通过`requestHostCallback`进行调度，给此方法传入一个回调`flushwork`,

> 本质上此函数由MessageChannel或setTimeout实现，根据是否是浏览器环境来用哪一种
>
> 具体可看SchedulerHostConfig.default.js

主要介绍浏览器环境下的`requestHostCallback`，即`MessageChannel`

通过`port.postMessage(null)`，通知调度中心开始调度，即启用`performUntilDeadline`，里面的逻辑主要是调用`scheduleHostCallback`(就是`flushWork`)，`flushWork`会开启一个工作循环`workLoop`，去执行任务，取出task，再根据`task.callback`去执行回调函数（即fiber树的构造），workLoop期间会根据是否还有时间执行task来进行是否中断任务，如果中断则返回true，到时在`performUntilDeadline`再去`port.postMessage(null)`去通知调度中心去开启另一个调度。



## 高优先级插队如何实现？

首先是要取消低优先级任务，调用`cancelCallback`取消任务，如果fiber构建过程中被中断，还要还原为初始化的fiber（`prepareFreshStack`），执行高优先级任务，最后`ensureRootIsScheduled`重新调度注册低优先级任务



# React的render阶段

## beginWork

mount：

1. 创建fiber节点
2. 设置fiber.flags标记

update：

1. 期间会根据**diff算法**判断是否能复用fiber节点
2. 设置fiber.flags标记
2. 处理更新队列`processUpdateQueue`

## completeWork

mount、update：

两阶段逻辑差不多

1. 创建DOM实例，fiber.stateNode指向此DOM实例
2. 给DOM节点设置属性，绑定事件
3. 期间也会设置fiber.flags标记
4. 上移副作用链表（effectList）
5. 判断当前fiber节点是否有副作用（fiber.flags标记），有则添加到父节点的effectList
5. 收集fiber.lanes到父节点的childLanes，一直到root



# React的commit阶段

遍历effectList去根据fiber操作DOM，将其渲染到页面或者移动等等

## beforemutation阶段

执行DOM操作前

- 调用getSnapshotBeforeUpdate

- 调用useEffect

  > useEffect异步执行的原因，防止阻塞浏览器的渲染

## mutation阶段

- 根据fiber.flags分类去进行处理
  - Placement
    - 获取父节点
    - 获取兄弟节点
    - 判断是否有兄弟节点进行appendChild或insertBefore操作
  - Update
  - Deletion
    - 调用componentWillUnmount生命周期，移除该DOM节点
    - 解绑ref
    - 调用useEffect销毁函数（即returnd的回调）
  - Hydrating
- 更新ref



## layout阶段

执行DOM操作后

- 执行生命周期componentDidMount、componentDidUpdate
- 赋值ref（毕竟此时DOM才展示在页面）
- setState如果有设置第二个参数此时就会执行



# React-Router v5

由三个包组成：

1. react-router，包含基础组件，以下两个包对其有依赖
2. react-router-dom通常用在浏览器项目
3. react-router-native通常用在react-native项目



## 实现原理

客户端路由实现的思想

- 基于hash的路由：通过监听`hashchange`事件，感知hash的变化
  - 可通过`location.hash`改变hash
  
- 基于H5 history路由
  - 改变url可以通过history.pushState和history.replaceState等，将url压入堆栈，同时应用`history.go()`等API
  
  - 通过事件`window.popState`监听url的变化，然后实现页面的切换
  
    > history.pushState、history.replaceState不会触发window.popState，通常前进、后退、调用history.back()、history.forward()、history.go()等api才会触发popState的方法



react-router实现的思想

- 基于history库去实现不同客户端的路由实现，并且保存历史记录，抹平浏览器的兼容问题。
- 通过维护一个路由路径列表，根据url的变化、设置路由路径去匹配Component，然后去渲染对应的页面



## 路由组件

- HashRouter，使用URL的hash属性（window.location.hash）实现路由的跳转，在路由加入#成为hash值，不会因为刷新而找不到页面
- BrowserRouter，浏览器使用的模式，H5使用的`historyAPI`。

## 匹配组件

- Switch，只会渲染第一个匹配的组件，如果没有设置path，则一定会被匹配，可通过此设置404页面

- Route，通过**path属性和浏览器location的pathname**进行匹配，匹配则返回内容，否则返回null

  > Route组件只匹配url的开头，不是整个匹配，也就是说不是精确匹配，解决方法，可以添加exact属性

## 链接组件

- Link，跳转到某页面，通过设置to属性
- NavLink，同Link组件，只不过会添加一些样式
- Redirect，重定向



## 路由传参

- param - match 参数显示在页面，刷新后不消失
- query - location 参数不显示在页面，刷新后消失
- state - location 参数不显示在页面，刷新后不消失



## Link标签和a标签的区别

Link标签、a标签都是标签，超链接，

Link是react-router里面实现路由跳转的链接，一般配合<Route>组件使用，Link标签的跳转不会刷新整个页面，只是实现匹配路由的页面内容刷新。

Link做的事情

- 有onclick就执行onclick
- 点击阻止a标签的默认事件
- 根据href（即ro），用history跳转，只是链接变了，不刷新页面，只实现匹配路由的页面内容刷新



a标签禁掉默认事件后如何实现跳转

```js
let domArr = document.getElementsByTagName('a');
[...domArr].forEach((item)=>{
    item.addEventListener('click',function(){
        location.href = this.href;
    })
})
```

> 最新为v6，有些组件有所改动，比如
>
> - Switch组件改为Routes
>
> - Route属性进行一些更新
>
> - Redirect组件改为Navigate
>
>   写成以下这种形式：<Route path="*" element={<Navigate to={} replace/>}></Route>



# Redux实现原理

一个状态管理工具

![image-20220207161324083](C:\Users\小浩王\AppData\Roaming\Typora\typora-user-images\image-20220207161324083.png)

纯函数：一个函数的返回结果只依赖于函数参数，执行过程没有副作用

三部分组成：Action（告诉store要进行什么操作）、Store（存储数据的地方）、Reducer（计算数据的地方）

- 组件内部将action分发（dispatch）给Store
- Store将state、action转发给Reducer，Reducer返回新state给Store
- 组件订阅Store的state，根据Store里的state变化进行View的渲染

常用方法

- `store.dispatch(action(state))`进行状态的改变

  ```js
  //具体形式
  store.dispatch({type:'',data:''});
  ```

- `createStore(reducer)`创建一个store对象
- `store.subscribe(()=>{})`监听事件，有变化就执行里面的方法
- `store.getState()`获取store里的state

**react-redux**常用方法

- `connect((state)=>{key:state的属性},{action里关于设置这个state的属性的方法集合})(UI组件)`

  > UI组件就是通常我们写的那个东西
  >
  > 这样就可以不用监听，connect会帮我们自动监听
  >
  > 调用也是直接调用那个集合里的方法，不用dispatch
  >
  > 这个connect（）返回的其实是个容器

> react-redux就是把redux的状态和react的UI绑定在一起，当你的状态改变，页面会自动变化，

**redux简单实现**

```js
const initState = {
    count: 0,
}

//纯函数
const reducer = (state = initState, action) => {
    switch (action.type) {
        case 'plus':
            return {
                ...state,
                count: state.count + 1
            };
        case 'subtract':
            return {
                ...state,
                count: state.count - 1
            };
        default:
            return initState;
    }
}

//核心
const createStore = (reducer) => {
    let currentState = {}; //公共状态
    let observers = [];
    
    // 获取值
    function getState() {
        return currentState;
    }

    // 分发，即设置，进行一系列操作
    function dispatch(action) {
        currentState = reducer(currentState, action);
        observers.forEach(cb => cb());
    }

    // 发布订阅
    function subscribe(fn) {
        observers.push(fn);
    }

    //初始化
    dispatch({ type: 'init' }); 

    return { getState, dispatch, subscribe };
}

//测试
const store = createStore(reducer);

console.log(store.getState());
store.subscribe(()=>{
    console.log('发生变化');
})
store.dispatch({ type: 'plus' });
console.log(store.getState());
```

**react-redux**实现原理

重点：Provider、connect

```js
import React, { Component, createContext } from 'react'

const StoreContext = createContext();

export class Provider extends Component {
    render() {
        return (
            <StoreContext.Provider value={this.props.store}>
                {this.props.children}
            </StoreContext.Provider>
        )
    }
}

export function connect(mapStateToProps, mapDispatchToProps) {
    return function (Component) {
        class Test extends React.Component {
            // React网上找最近的provider,然后使用其值
            static contextType = StoreContext;

            componentDidMount() {
                console.log(this.context);
                // 订阅，【重点】，否则不更新
                this.context.subscribe(()=>{
                    this.forceUpdate();
                })
            }

            render() {
                return (
                    <Component
                        {...this.props}
                        {...mapStateToProps(this.context.getState())}
                        {...mapDispatchToProps(this.context.dispatch)}
                    ></Component>
                )
            }
        }

        return Test;
    }
}
```



# Hook

钩子，搭配函数组件

大概相关结构

```js
update = {
    action, //需要进行的操作
    next, //下一个update对象
}

hook = {
    memoziedState: null, //最新的值
    next, //指向下一个hook
    queue:{
        pending, //指向最新的update对象 
    }
}

fiber = {
	memozied,
}
```

每个组件对应一个fiber，fiber里有个memoizedState（单链表），存储着对应的hook，hook里面有个queue属性（循环链表），存储着update对象。

## 为什么要用链表存储所有更新操作，而不存储最新的操作

因为每个操作会依赖前面操作的值，所以需遍历链表去进行对应的操作

## 为什么这样设计循环链表

因为update会涉及到优先级之类，优先处理用户输入这类事件，所以设计成循环链表，到时先处理此任务，到时再去执行其他任务



## hook相关api

- useState，处理状态

  ```js
  const [age.setAge] = useState(0);
  setAge(age+1);
  ```

- useEffect，处理副作用，有点类似`componentDidMount、componentDidUpdate`

  ```js
  //相当于componentDidMount
  useEffect(()=>{
      
  },[]); 
  //此[]相当于监测数组，啥都不写表示谁都不监测
  
  //相当于componentDidUpdate
  useEffect(()=>{
  
  });
  ```

  

- useRef，标记组件，可以获取其DOM属性

  ```js
  function(){
  	const input = useRef(null);
      
      //之后可以用input.current获取其DOM属性
      
      return (
      	<>
          	<input ref={input}/>
          </>
      )
  }
  ```

- useContext，处理跨层级传值，直接获取其值，不需用Consumer组件，方便很多

  ```js
  const value = useContext(context);
  ```

  

- useMemo，性能优化，缓存值，依赖于value，只有当value变化时，才会重新执行函数

  ```js
  const func = useMemo(()=>{
      return 'test';
  },[value]);
  ```

  

- useCallback，性能优化，缓存回调函数，依赖value，只有当value变化时，才会重新定义函数

  ```js
  const func = useCallback(()=>{
      console.log('callbacl');
  },[value]);
  ```

Hooks组件模拟生命周期

| Hooks组件               | 类式组件                 |
| ----------------------- | ------------------------ |
| useState                | constructor              |
| useState里的update函数  | getDerivedStateFromProps |
| useMemo                 | shouldComponentUpdate    |
| 函数本身                | render                   |
| useEffect传入空数组     | componentDidMount        |
| useEffect不传数组       | componentDidUpdate       |
| useEffect里面返回的函数 | componentWillUnMount     |



