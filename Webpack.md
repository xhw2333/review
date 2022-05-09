# Webpack

## 简介

本质上是一个node程序，可以说是一个静态模块的打包工具。

作用是分析你的项目，找到js模块以及一些浏览器不能直接运行的拓展语言（scss、ts），为其打包成合适的格式给浏览器使用。

webpack的核心是**负责编译的complier和负责构建捆绑包的compilation**



## 核心概念

- entry：入口
- module：模块，在webpack中，一切皆为模块，一个模块对应一个文件
- chunk：代码块，
- bunble：chunk的打包产物，通常一个bunble对应一个chunk
- loader：模块转换器，将非js模块转换为webpack能够识别的js模块
- plugin：拓展插件，在webpack运行的各个阶段，都会广播出去对应的事件，插件可以监听到事件的发生，在特定时机执行对应的事。
- output：出口



## 工作流程

1. 解析参数，合并命令行和`webpack.config.js`的参数，得到配置项

2. 开始编译：根据上面得到的配置项去初始化compiler对象，调用`compiler.run`进行编译（在编译的第一阶段是`compilation`，会为不同module注册好对应的factory）。

   > compilation可以访问到所有模块以及其依赖

3. 编译模块：可以说分为两步

   - 调用loader去对模块进行转化，生成webpack能够识别的js模块
   - 调用AST识别工具（acorn）对js代码进行语法分析，生成AST抽象语法树，找到依赖，继续对依赖的模块进行loader转化、AST解析操作，如此递归下去，直到解析完毕

4. 编译完毕：得到模块依赖树，生成chunk代码块。

5. 输出资源：对chunk进行打包，输出到文件系统中。



## loader

作用：处理任意类型的文件，将他们转化为可以让webpack处理、识别的有效模块

### 调用顺序

从右往左调用，参考css-loader先执行、再执行style-loader。

### loader原理

**每一个loader本质上是个函数**，对输入参数的内容进行转义然后返回出来

返回方式有三种

- 直接return，一般是loader只有一个处理结果
- this.callback，一般是loader有多个处理结果
- 用this.async来创建callback，用来创建异步loader

loader输入：文件字符串，上一个loader转化的结果

loader输出：文件字符串，source map， AST对象



### 常见loader

- vue-loader，处理.vue文件，将template、js、样式进行拆分
- style-loader，将css-loader处理的结果转化为一段js代码，这段代码可以将css插入到html中
- css-loader，将css压缩为js代码，并存放到一个数组里，让style-loader去处理



## plugin

插件就是用来拓展一些webpack的功能，比如代码压缩。在webpack运行的各个阶段，会广播出对应的事件，插件可以监听到这些事件，在特定的时机执行对应的操作。

### 常见plugin

- uglifyjs-webpack-plugin：压缩js代码，配合tree-shaking使用
- split-chunks-plugin：代码分片。webpack4之前使用的是`common-chunk-plugin`

### 编写plugin

plugin主要由以下部分组成

- 一个js方法或js类
- 其原型上需要定义一个叫做`apply`的方法
- 注册一个事件钩子
- 操作webpack内部实例特定的数据
- 功能完成后，调用webpack提供的回调

```js
//最简单的plugin
class MyPlugin{
    //执行apply注册该插件
    apply(compiler){
        compier.hooks.done.tap('My Plugin',compilation=>{
            console.log('plugin');
        })
    }
}
```



## tree-shaking

tree-shaking可以**检测项目里没有被引用的模块（称为”死代码“）**，然后在打包构建时移除它。其实tree-shaking只是给死代码打上标记，真正移除需要靠压缩工具比如**uglifyjs-webpack-plugin（压缩js代码）**来实现

实现tree-shaking的前提

- 基于es6模块，不能解析commonjs和amd等规范的模块
- 如果有babel-loader、typescript的设置，要禁用，因为会转换为es5语法



## complier和compilation的区别

complier：

- 继承于tapable类
- webpack刚开始创建就存在了，通过配置项初始化而来，所以通过他可以访问到配置

compilation：

- 继承于tapable类
- 在`compiler.run`（准备编译）时创建
- 可以访问到所有模块及其依赖



## loader和plugin的区别

loader，webpack只能读取js文件，不能读取css、png这些文件，所以需要调用loader来进行转化

plugin，拓展一些webpack不能实现的功能，在webpack运行的各个阶段，都会广播出对应的事件，插件可以监听到事件的发生，在特定的时机执行对应的事情，比如说代码分割



## 模块加载

- commonjs是运行时加载，就是在运行时才将整个模块引入

- es module是编译时加载好，运行时直接在缓存里引入即可



## 热更新HMR

**Hot Module Replacement**，简称HMR，无需完全刷新页面，实现更新。

基于`webpack-dev-server`的模块热替换，实现局部刷新，保持状态，比如：输入框的输入等。

大致原理：

- **`webpack-dev-server`**通过**express**创建一个本地服务器，与浏览器通过websocket连接，如果本地资源有修改，`webpack-dev-server`向浏览器推送更新事件，并附带此次更新资源的hash值
- 浏览器通过判断hash是否需要拉取资源，如果需要拉取，则向`webpack-dev-server`发起资源拉取请求，一般先拉取**文件更新列表**，再**根据此去拉取更详细的更新增量**。

