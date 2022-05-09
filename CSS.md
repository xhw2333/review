# 文档流

相对于元素来说，元素从上到下排列，从左到右排列

# 文本流

相对于文本来说，文本从左到右排列，从上到下排列



# 盒子模型

盒子模型主要由content、padding、border、margin组成

- 标准盒子模型
  - 宽=content
  - box-sizing：content-box；
- 怪异盒子模型（IE盒子模型）
  - 宽=content+padding+border
  - box-sizing：border-box；
    

# li与li之间的空白间隙？

浏览器会将inline元素间的空白字符（空白、换行等）渲染成一个空格，因为平时把li标签换行了。

**解决**：

- ul设置font-size为0，li重设置字体，
- 使用float
- 写在一行，太丑了



# 替换元素和非替换元素

- 替换元素就是浏览器根据元素属性去显示具体内容的元素，且拥有固定宽高。
  - 例如：img依靠src，inpupt依靠type来显示不同的框
  - 常见替换元素为img、input

- 非替换元素就是直接显示内容的元素，例如div



# 判断元素到达可视区

**window.innerHeight**：表示浏览器的可视化高度

**document.body.scrollTop || document.documentElement.scrollTop**：浏览器滚动过的距离

**元素.offsetTop**：元素距离文档顶部的距离

进入可视区：**元素.offsetTop < window.innerHeight+document.body.scrollTop**

![image-20220228153937855](C:\Users\小浩王\AppData\Roaming\Typora\typora-user-images\image-20220228153937855.png)



# 长度单位

- px：像素，固定单位
- em：相对于父元素的font-size的倍数，父元素没有设置font-size，则默认为16px
- rem：相对于根元素（html）的font-size的倍数
- 百分比：子元素百分比直接相对于父元素
- vw、vh：视图窗口的宽高



# BFC

块级格式化上下文，简单来说就是一个隔离的区域，不会对外界产生影响

触发条件

- `overflow`值不为visible
- `float`值不为none
- `display`值为`flex、table-cell、inline-block`等等
- `position`不为`static、relative`等

解决问题

- 外边距塌陷、重叠
- 浮动覆盖问题



# position

- static：默认值
- absolute：绝对定位，相对于最近的position值不为static的祖先元素来定位，脱离文档流
- relative：相对定位，相对于元素原来位置来定位，没脱离文档流
- fixed：固定定位，相对于屏幕来定位，脱离文档流
- sticky：粘性定位，相对定位和固定定位的组合，跨越阈值前为相对定位，跨越阈值后为固定定位



# 七阶层叠

层级上下文：**html一个三维的概念。元素发生了堆叠**，每个盒子模型的位置都是三维的，有平面画布上的x轴和y轴，还有表示层叠的z轴。

层叠等级：**在Z轴上的位置**

层叠顺序从内往外

1. background/border
2. z-index为负值
3. block块级元素
4. float元素
5. inline/inline-block元素
6. z-index：0/z-index：auto元素
7. z-index为正值的元素

> z-index仅在定位为非static的元素才有效



# 垂直居中

单行文本：设置line-height=height

多行文本：设置`display:tabel-cell;vertical-align:middle`

绝对定位+transform



# flex布局

- 设置了`display:flex | inline-flex`，这个元素就变成了一个flex容器
- flex容器有**水平主轴（main axis）和垂直的交叉轴（cross axis）**
- 每个单元块称为flex-item，每个项目占据的主轴空间为**main size**，占据的交叉轴空间为**cross size**
- **flex-direction**决定主轴的方向，默认row（水平），还可以设置column（垂直）
- **justify-content**规定项目在主轴的对齐方式
- **align-items**规定项目在交叉轴的对齐方式
- 关于**flex属性**
  - flex为**gsb**的集合：**flex-grow、flex-shrink、flex-basis**
  - flex-grow：有剩余空间，项目的放大比例，默认值0
  - flex-shrink：无剩余空间，项目的缩小比例，默认值1
  - flex-basis：项目所占的主轴空间，默认值auto
- `flex:1`表示`1 1 auto`，所有项目等比例缩扩



# 优先级

- id选择器
- 类选择器、属性选择器、伪类选择器
- 标签选择器、伪元素选择器

> css选择符从右往左进行匹配



# 伪元素和伪类的区别

- 写法不一样，伪元素规定需要两个冒号，伪类需要一个冒号
- **存在**区别
  - 伪元素是创建了一个新元素，逻辑存在但实际不存在DOM文档中，即js获取不到他，源码看不见，但外部可显示
  - 伪类是基于存在的元素，根据特征，去进行操作，通俗理解就是他是存在但你看不到



# 浮动

设置了`float`，元素会浮动，父元素无法被撑开

**清除浮动**

- 利用父元素的伪元素after进行清除

  ```css
  father::after{
      content: '';
      display: 'block';
      clear: both;
  }
  ```

  > clear只在块级元素中起作用

