# Vue

梳理一些重要知识

## MVVM

MVC的改进版，通常MVC是指Model（模型）、View（视图）、Controller（控制器），MVVM就是在此基础上多了个VM（view-model），这个层主要是view和model之间的桥梁，实现双向数据绑定，本来controller是控制业务逻辑和数据转化的，然后因为代码太复杂、堆在一起，所以最终把负责业务逻辑的代码抽离出来，然后由VM来负责管理，形成MVVM模式。

比如：数据的双向绑定v-model，data变，view也会变，view变，data也会变

原理：`Object.defineProperty`



# VueX

一种状态管理的工具

- state -- 存放公共数据，组件里`this.$store.state.变量名`
- mutation -- 同步方法，更新state里的数据
- action -- 异步方法，组件通过调用`this.$store.dispatch('方法名'，参数)`
- getter -- 读取state里的数据
- commit -- 状态提交，对mutation操作进行提交
- 页面刷新，vuex里的数据会消失，数据持久化可以采用
  - localStorage
  - persistedstate插件

