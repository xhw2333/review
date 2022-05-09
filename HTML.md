# src和href的区别

src：指向外部资源的位置，会将资源下载并占位到当前标签所在位置，应用到页面中

href：一个外部资源的链接，访问会跳转其所在的网页



# doctype（文档类型）的作用

html5中的文档类型声明，告诉浏览器该以什么样的文档类型来解析该文档

```html
<!DOCTYPE html>
进入html5标准模式，不写则是混杂模式
```

浏览器渲染页面的两种形式

- 标准模式
- 怪异模式（混杂模式）



# head标签

用于定义头部，描述文档各种属性和信息，可以引用css样式表，js脚本等



# meta标签

一种head标签的辅助性标签，就是对整个文档的描述

由name和content定义

**作用**

- charset，描述html文档的编码类型
- viewport，适配移动端



# html5的更新

- 语义化标签
- dom操作
  - document.querySelector()
  - document.querySelectorAll()
- history API



# 行内元素和块级元素

行内元素：a、span

块级元素：div、p



# 网页乱码？

网页源代码默认编码GBK，内容中的中文是UTF-8

所以设置`<meta charset="utf-8"/>`

