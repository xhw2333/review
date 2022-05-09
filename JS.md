# js数据类型

- 基本类型：Number类型、String类型、Boolean类型、null、undefined，Symbol类型
- 引用类型：Object类型，Function类型、Array类型

基础类型的值和引用类型的地址放在栈里，引用类型的值放在堆里

## 类型判断

- typeof
  - 可以区分基本数据类型
  - typeof null等于Object，typeof Array等于Object，不能精确区分object、array、null类型
  
- instanceof
  - 用法`object instanceof constructor`
  - 检测`constructor.prototype`是否存在于参数`object`的原型链上
  - [] instanceof Array 等于 true
  - null instanceof Object 等于false
  - 精确区分Array、Object、Function
  
  **instanceof原生实现**
  
  ```js
  function myinstanceof(a,b){
  
      while(a !== null){
          let proto = Object.getPrototypeOf(a);
          if(proto === b.prototype) return true;
          a = proto;
      }
  
      return false;
  }
  
  console.log(myinstanceof({},Object)); // true
  console.log({} instanceof Object); //true
  ```
  
- constructor
  
  - `(1).constructor === Number; //true`
  
    > **注**：constructor不在1本身上，会去1的原型上找constructor属性
  
- 借用Object.prototype.toString
  - 精确区分
  - Object.prototype.toString.call(1) 返回’[object Number]‘
  - Object.prototype.toString.call(null) 返回’[object Null]‘
  - Object.prototype.toString.call(undefined) 返回’[object Undefined]‘

## 类型的比较（即==）

- 对象与基础类型比较：对象一般都是先转为字符串（调用valueOf，toString），再转数字，再进行比较
- 布尔与数字比较，布尔会转为数字再比较

- undefined、null遇到\=\=不会发生隐式类型转换

  > 如何解释**undefined\=\=null**？
  >
  > js规定这两个数相等，但不能进行隐式类型转换

- boolean值转换，除了null、undefined、NaN、0、""转为false,其他的都会转化为true

  ```js
  ![] == []; //true
  //解释：![]会转为false，[]与false比较，[]会这样子转化：[] -> ‘’ -> 0，false转为0
  
  [] == false; //true
  ```

**总结：两边都尽量转换为数字进行比较**

## +操作符

- 两边至少有一个字符串，则都会转化为字符串然后进行运算

  ```js
  1 + '23' === '123'; //t
  ```

  

- 其他情况，转化为数字进行运算



# 执行上下文和执行栈

## 执行上下文

​	关于执行上下文，其实就是js代码**执行和解析**的环境

执行上下文分为3种类型

1. 全局执行上下文，整个js文件执行的执行上下文
2. 函数执行上下文，每当函数被调用时，就会创建一个函数执行上下文
3. eval执行上下文

执行上下文有**三个重要属性**

- 变量对象（Variable Object）
- 作用域链（Scope Chain）
- this

## 执行栈

​	一个存储执行上下文的栈，遵循先进后出。

​	js引擎第一次遇到脚本，创建一个全局执行上下文并推入栈中，每当调用一个函数，创建一个函数执行上下文并推入栈中，执行完毕，弹出来。

## 创建执行上下文

创建执行上下文主要包括两个阶段

1. 创建阶段
2. 执行阶段

### 创建阶段

创建阶段主要由三部分组成

1. this绑定
2. 词法环境
3. 变量环境

#### this绑定

- 全局执行上下文：this指向全局对象（即window），严格模式模式下为undefined
- 函数执行上下文：this指向调用此函数的对象。

#### 词法环境

词法环境是一个标识符--变量映射（变量名--实际对象）的结构。

简单来说词法环境就是自身+外部环境，由以下两部分组成

- 环境记录器：实际存储函数和变量的地方
- 外部环境的引用：访问其外部环境的引用

词法环境有两种类型

1. 全局环境：拥有全局对象（window）、关联的对象和属性以及用户自定义的变量，存储外部环境引用为null。
2. 函数环境：环境记录器中记录用户自定义的变量，包含`arguments`对象，外部环境可以是全局环境或者外部函数的环境

环境记录器有两种类型

1. 声明式环境记录器，用于函数词法环境，记录变量、函数、参数
2. 对象记录器，用于全局词法环境，用来定义出现在全局上下文的变量和函数的关系

#### 变量环境

其实就是一个词法环境，es6中的区别是

- 词法环境存储函数声明和变量（`let、const`）的绑定
- 变量环境存储变量（`var`）的绑定

### 执行阶段

此阶段完成对变量的分配，最后执行代码

# 变量对象

关于变量对象

- 全局执行上下文的变量对象就是全局对象
- 函数执行上下文的变量对象

主要提一下**函数执行上下文的变量对象**

它也被称为**活动对象**，因为进入函数执行上下文才有此变量对象，才被激活（该对象上的属性可以被访问） ，所以被称为**活动对象**

举一个例子

```js
function func(a){
    var b = 1;
    var d = function(){console.log('d');}
    function c(){console.log('c');}
}
func(1);
```

进入执行上下文时

```js
AO:{
    a: 1,
    b: undefined,
    c: function(){console.log('c');},
    d: undefined,
}
```

执行完代码

```js
AO:{
    a:1,
    b:undefined,
    c:function(){console.log('c');},
    d:function(){console.log('d');}
}
```

看完以上例子，总结一下：

**进入函数执行上下文**，变量对象包括：

- 函数形参，无实参时值为`undefined`
- 函数声明，变量对象存在相同名称的属性，则替换它
- 变量声明，变量对象存在相同名称的形参和函数，则不会影响（*），没有声明但赋值为成为全局对象

> 进入函数执行上下文，处理顺序是==形参赋值->函数声明->变量声明->其他操作（赋值等等）==



# 作用域与作用域链

作用域：程序源代码中定义变量的区域

js采用的是**词法作用域**，也就是**静态作用域**，函数作用域在函数定义时就决定了，**函数作用域基于函数创建的位置**。

> 动态作用域：函数被调用时，作用域才被决定。

作用域链：当找变量时，会先在当前执行上下文的变量对象找，如果找不到，就会在其父级（词法层面上）的执行上下文的变量对象找，一直找到全局执行上下文的变量对象。这样由多个执行上下文的变量对象组成的链表就形成了**作用域链**。

函数创建：会形成作用域链是取决于**函数内部的[[scope]]属性**，创建函数时会将其父级（到全局）的变量对象保存到**[[scope]]**属性中。此时scope并不能代表完整的作用域链，只是包括父级的层级链

函数激活：当函数被调用，被argument对象初始化，创建VO/AO对象，将当前执行上下文的变量对象添加到**作用域链**的前端，最终完整的作用域链形成。



# 函数传参

函数传参本质上是值的传递，基本类型传的是基本类型的值，引用类型传递的是指针的地址

```js
let obj = {a:1};

function func(obj){
    obj = {};
}
func(obj);
console.log(obj); //{a:1}
```



# 基本包装类型

定义一个基础类型，后台会创建一个对应的基本包装类型对象，所以基本类型才可以调用方法



# 闭包

闭包指有权访问另一个函数中作用域变量的函数。

一般来说，函数创建时会建立一个活动对象（AO），当函数执行完，此活动对象就会被销毁。但因为闭包，保存着对另一个函数内变量的引用，所以那个函数的活动对象不会被销毁。

原理：作用域链

闭包作用

- 形成私有变量，保护其不受污染
- 保存私有变量的值

闭包缺点

- 内存泄漏

如何解决内存泄漏

- 在不使用变量时，将其设置为null或者删除此变量



# this指向

- 全局执行上下文，this指向全局对象
- 函数执行上下文，this指向调用该函数的对象
- 对于DOM事件处理函数，指向触发事件的元素，绑定该事件的DOM节点
- 箭头函数，this指向外层作用域

总结：this指向调用该方法的对象，没明确调用的话非严格模式下会指向window，严格模式下会是undefined



# apply、call、bind

- 三者都是用来改变this指向
- 三者第一个参数是改变this指向的对象
- apply、call会立即调用函数，bind返回一个this绑定的函数，可稍后执行
- apply传入的其他参数是参数列表，就一个数组嘛，call传入时需要把参数全部列出，bind在需要传参时也是列出所有参数
- 传入的this为undefined或null，改变后的this指向window

apply原生实现

```js
Function.prototype.myapply = function(context){
    // 传入的context为空，那其指向window
    context = context || window;
    
    //this指向调用这个myapply方法的对象，其实这里就是原来那个函数，context为要改变this后的对象
    //fn有可能context本身存在该值，所以可能Symbol唯一标识替代
    const fn = Symbol();
    context[fn] = this;
    
    //如果不用扩展符的话，可以用eval
    //生成这样类型str = 'arguments[1],arguments[2]'的字符串
    //eval('context.fn(str)');
    
	// 函数返回的结果
	let res;

	if (arguments.length > 1) {
        // 判断有无传入其他参数
		const args = arguments[1];
		res = context[fn](...args);
	} else {
		res = context[fn]();
	}
    
    //删除此属性，不然context会存在该属性
    delete context.fn; 
    
    return res;
}
```

call原生实现

```js
Function.prototype.mycall = function (context) {
    console.log(this, arguments);
    
    // 传入的context为空，那其指向window
    context = context || window;
    
    console.dir(context);

    // 控制fn的唯一性
    const fn = Symbol();
    context[fn] = this;

    // 函数返回的结果
    let res;

    // 判断有无传入其他参数
    const args = [];
    for(let i = 1; i < arguments.length; i++){
        args.push(arguments[i]);
    }

    if (args.length > 0) {
        res = context[fn](...args);
    } else {
        res = context[fn]();
    }

    // 删除属性
    delete context[fn];

    return res;
}
```

bind原生实现

```js
Function.prototype.mybind = function(){
    let that = this;
const args = Array.prototype.slice.call(arguments, 1);
	const F = function(){};
	F.prototype = this.prototype;
	let func =  function () {
    	let fn = Symbol();
    	context = context || window;
    	context[fn] = that;

    	const finalArr = args.concat(Array.from(arguments));

    	let res = context[fn](...finalArr);

    	delete context[fn];

    	return res;
	}
    func.prototype = new F();
	return func;
}
```

使用apply实现bind

```js
// 使用apply实现的bind
Function.prototype.applybind = function (context) {

    let that = this;
    let args = Array.prototype.slice.call(arguments,1);
    const F = function(){};
    F.prototype = this.prototype;
    let func =  function(){
        return that.apply(context,args.concat(Array.from(arguments)));
    }
    func.protoype = new F();
    return func;
}
```



# 对象

`Object.defineProperty`，定义属性的一些特性，比如可写（writable），可遍历（enumerable），可删除、重新定义（configurable），get、set

```js
let person = {};
Object.defineProperty(person,'name',{
    writable:false,
    enumerable: false,
    configurable: false,
    value:'xhw',
})
```

> get和value只能用一个
>
> set和writable只能用一个



## 创建对象的模式

1. 工厂模式，没有解决标识符问题，即该对象是什么特定类型
2. 构造函数模式，创建新实例时方法都会被创建一次
3. 原型模式，属性、方法都会被所有实例共享
4. 组合模式，属性定义在构造函数，方法定义在原型上，

## for...in循环

可遍历对象，会遍历的整个原型链



# 原型和继承

> **\_proto\_**和**constructor**是对象独有的
>
> **prototype**是函数独有的
>
> 函数是对象，所以也拥有**\_proto\_、constructor**属性

- **\_proto\_**是一个对象指向另一个对象

- **constructor**是一个对象指向一个函数，这个函数为该对象的构造函数

- 函数创建时，他就有一个**prototype**属性，指向其原型

- 最简单的一个例子，一个构造函数Person，实例p（用**构造函数Person**实例化）

  关系如下：

  - p的隐式原型\_proto\_，Person的显式原型prototype。两者都是指向同一个对象
  - 原型的constructor属性指向构造函数Person

  ```js
  function Person(){}
  
  const p = new Person();
  
  Person.prototype === p._proto_; //true
  Person.prototype.constructor === Person; //true
  ```

- 所有对象都是通过Object实例化而来的，所以Object本质也是个构造函数，是由Function实例化而来的（可能有点绕，细看）。

- ps：

  - `Function.prototype === Function._proto_`
  - `Object.prototype._proto_ === null`

​	![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/10/31/16e1f9f4a315ac95~tplv-t2oaga2asx-watermark.awebp)

其中 `constructor` 属性，**虚线表示继承而来的 constructor 属性**。

## 原型链

在此对象找不到对应的属性，就会往其原型上去找，如果还是找不到就其原型上的原型去找，这样就构成了原型链。

## new方法

通常new一个构造函数来生成一个实例会经历以下步骤

1. 创建一个对象
2. 为其添加**\_proto\_属性**，将该属性链接至构造函数的原型
3. 将对象其作为this的上下文，执行构造函数，为该对象添加属性和方法（**改变构造函数的this指向**）
4. 判断构造函数的结果是否为对象，是的话则返回执行结果，否则返回创建的对象

new原生实现

```js
function mynew(){ 
    const Con = Array.prototype.shift.call(arguments); //Con为传进来的构造函数
    //
    const obj = new Object(); //新创建的对象
    obj._proto_ = Con.prototype;
    //以上两行代码可以换成
    //const obj = Object.create(Con.prototype);
    
    const res = Con.apply(obj,arguments);
    
    return typeof res === 'object'?res:obj; //返回一个对象
}

function Person(name){
    this.name = name;
}

const p = mynew(Person,'xhw');
console.log(p);
```

> `Object.create(proto)`，参数proto--新创建对象的原型对象
>
> 返回一个新对象，带着指定的原型对象和属性，**简单来说就是此对象的原型指向proto**



## 继承

- 原型链继承

  - 关键步骤：子类的原型指向父类的实例，`Sub.prototype = new Super()`相当于`Sub.prototype._proto_ = Super.prototype`
  - 问题：
    - 子类的实例会共享父类上的属性（因为原型上的属性会被所有实例共享，父类实例此时作为子类的原型）
    - 不能向父类传参数，即父类构造函数的属性不能初始化

  ```js
  function Super(){
      this.age = 100;
      this.skill = ['eat','drink'];
  }
  
  Super.prototype.say = function(){
      console.log(`I am ${this.name}.I am ${this.age}.I can ${this.skill.toString()}`);
  }
  
  function Sub(name){
      this.name = name;
  }
  
  Sub.prototype = new Super(); //相当于Sub.prototype._proto_ = Super.prototype;
  //此时Sub.prototype的constrcutor属性已经不再指向Sub构造函数，因为constrcutor属性已经不在这个Sub.prototype对象上了，只能在Super.prototype对象上找到constructor属性
  //此句可以证明以上结论 Sub.prototype.hasOwnProperty('constructor') === false
  
  const a = new Sub('A');
  const b = new Sub('B');
  
  a.skill.push('speak');
  a.say(); //I am A.I am undefined.I can eat,drink,speak
  b.say(); // I am B.I am undefined.I can eat,drink,speak
  a.age; //100
  b.age; //100
  ```

  

- 借用构造函数继承

  - 核心：使用**call**方法去继承父类
  - 缺点：**不能继承父类原型上的属性和方法**

  ```js
  function Super(age){
      this.age = age;
  }
  
  Super.prototype.say = function(){
      console.log(`I am ${this.name},I am ${this.age}`);
  }
  
  function Sub(name,age){
      Super.call(this,age);
      this.name = name;
  }
  
  const a = new Sub('A',12);
  const b = new Sub('B',13);
  
  a.age; //12
  b.age; //13
  
  a.say(); //报错
  b.say(); //报错
  ```

  

- 组合式继承

  - 核心：使用call方法继承父类，以及让子类的原型指向父类的实例
  - 缺点：二次调用父类构造函数，性能问题。

  ```js
  function Super(age){
      this.age = age;
      this.skill = ['eat','drink'];
  }
  
  Super.prototype.say = function(){
      console.log(`I am ${this.name},I am ${this.age},I can ${this.skill.toString()}`);
  }
  
  function Sub(name,age){
      Super.call(this,age);
      this.name = name;
  }
  
  Sub.prototype = new Super();
  Sub.prototype.constrcutor = Sub;
  
  const a = new Sub('A',12);
  const b = new Sub('B',13);
  
  a.skill.push('speak');
  a.say(); //I am A,I am 12,I can eat,drink,speak
  b.say(); //I am B,I am 13,I can eat,drink
  ```

  

- 原型式继承

  - 核心：创建一个构造函数，其原型指向父类，返回一个实例
  - 缺点：子类的实例会共享父类上的属性和方法

  ```js
  function createObj(o){
      function F(){};
      F.prototype = o;
      return new F();
  }
  
  let person = {
      name: '***',
      age: '22',
      skill: ['eat','drink'],
  }
  
  const a = createObj(person);
  a.name = 'nidie';
  a.skill.push('speak');
  
  const b = createObj(person);
  b.name; //***
  b.skill;  //['eat', 'drink', 'speak']
  ```

  

- 寄生式继承

  - 在原型式继承上进行二次封装，再利用原型式继承里返回的实例进行属性、方法的添加，再返回此实例。（增强对象）
  - 缺点：很明显，代码不能复用，

  ```js
  function createAnother(o){
      const obj = createObj(o);
      obj.name = 'nidie';
      obj.say = function(){
          console.log(this.name,this.age,this.skill.toString());
      }
      return obj;
  }
  
  function createObj(o){
      function F(){};
      F.prototype = o;
      return new F();
  }
  
  let person = {
      name: '***',
      age: '22',
      skill: ['eat','drink'],
  }
  
  const a = createAnother(person);
  ```

  

- 寄生组合式继承

  - 解决组合继承性能上的问题
  
  ```js
  function createObj(o){
      function F(){};
      F.prototype = o;
      return new F();
  }
  
  function Super(age){
      this.age = age;
      this.skill = ['eat','drink'];
  }
  
  Super.prototype.say = function(){
      console.log(this.name,this.age,this.skill.toString());
  }
  
  function Sub(name,age){
      Super.call(this.age); //将父类的属性复制到此
      this.name = name;
  }
  
  //创建一个父类副本，继承父类原型上的属性和方法
  const p = createObj(Super.prototype);
  //设置子类原型
  Sub.prototype = p;
  //修改父类副本的constructor属性，使其指向子类
  p.constructor = Sub;
  
  const a = new Sub('A',12);
  const b = new Sub('B',11);
  a.skill.push('speak');
  
  a.skill; //['eat', 'drink', 'speak']
  b.skill; //['eat', 'drink']
  ```



> es5的继承是先创建子类实例，然后将父类属性复制到实例上

## es6的class

继承使用`super()`相当于`父类.call(this)`（**有争议**），因为子类中没有this，是继承父类的this



# 函数式编程

## 偏函数

固定某些函数参数，返回一个新的函数，用于接收其他参数

## 柯里化

将多个参数的函数转为单参数的函数

```js
function currying(fn){
	const arg = [];
    
    return function cb(){
        if(arguments.length === 0){
			return fn.apply(this,arg);
        }
        
        arg.push(...arguments);
        return cb;
    }
}

function add(a,b,c){
    return a + b + c;
}

const test = currying(add);

test(1)(2)(3)(); 
```



# 浅拷贝和深拷贝

浅拷贝：只是对拷贝对象的第一层

> Object.assign和扩展运算符都是浅拷贝

深拷贝：完全拷贝一份新的对象

```js
//map解决循环引用问题
function deepClone(obj,map = new Map()){
    //判断s数组
    const objClone = Array.isArray(obj)?[]:{};
    if(obj && typeof obj === 'object'){
        if(map.has(obj)){
            return map.get(obj);
        }
        map.set(obj,objClone);
        for(const key in obj){
			if(obj.hasOwnProperty(key)){
                if(obj[key] && typeof obj[key] === 'object'){
                    objClone[key] = deepClone(obj[key],map);
                } else {
                    objClone[key] = obj[key];
                }
            }
        }
    }
    
    return objClone;
}
```



# 垃圾回收机制

js引擎找出不再继续使用的那些变量，释放其占用的内存，垃圾回收器按**固定的时间间隔周期性**地执行这一操作

## 引用计数

记录每个值被引用的次数，如果被引用了，就加1，释放了就减1，当值为0，表示这个值不再被引用，到时垃圾回收器就会释放其占用的内存空间                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                

## 标记清除

当变量进入执行环境时，标记为“进入环境”，离开执行环境时，标记为“离开环境”，最后垃圾回收器会销毁掉标记为“离开环境”的变量，并回收其占用的内存



工作流程：

1. 垃圾收集器在运行时将在内存中的**所有变量加上标记**
2. **从根部出发**，将**能访问到的对象**的标记**清除**
3. 最后垃圾收集器会执行内存清除工作，销毁那些带标记的值，并回收其占用的内存空间

> 总的道理就是要**将从根部无法到达的对象标记，然后清除标记的对象**

 

缺点：会产生外部碎片



ps：而标记整理可以解决此问题，将活动对象移到内存一侧，清理掉非活动对象



## V8对GC的优化

**采用分代式垃圾回收**

分为新生代和老生代对象，将堆内存划分为两个空间

### scavenage算法

用于新生代对象，就是将新生代空间一分为二，一个作为使用区（一般存储着对象），一个作为空闲区，回收将活动对象复制到空闲区，清理掉使用区的对象，然后两者交换（即空闲区作为使用区，使用区作为空闲区）

晋升：新生代对象变为老生代对象 

> 条件：在对象复制到空闲区时做一次判断，该对象是否已经经历过一次scavenge算法，如果是，直接晋升



### 增量标记和惰性清理

用于老生代 ，

#### 增量标记

就是将垃圾回收标记这个任务分成一小步一小步来完成。来避免“全停顿”这种现象。

#### 如何恢复和暂停？

v8采用了**三色标记法**来实现，

规定如下：

白色：未被标记的对象

灰色：本身被标记，成员变量未被标记

黑色：本身、成员变量被标记



**暂停直接退出这个阶段即可**

**恢复**：通过判断内存中有无灰色标记的对象来判断标记是否完成？没有灰色则进入清理阶段，有则从灰色节点继续执行



#### 那如果运行期间修改了引用，这又怎么处理？

比如上一次增量标记阶段，A->B->C被标上黑色，然后在js运行阶段，B下新增一个引用D（D是新增的，所以没有标记，即白色），然后在下一次增量标记阶段，因为没有灰色，所以进入清理阶段，但这样是不对的，毕竟D是可以被访问到的，是活动对象。所以v8使用了**写屏障机制**，规定**一旦有黑色标记加入引用对象，那么该引用对象被标记为灰色**



#### 惰性清理

增量标记只是标记，清理是交给惰性清理来完成的



# 事件循环

js是一种单线程的语言，事件循环是为了解决js单线程不会阻塞的一种机制。

明白是由三个部分来工作：

- 调用栈。任务等待着主线程来执行，执行完就弹出，后进先出。
- 宏任务队列。存放宏任务，遵循先进先出。
- 微任务队列。存放微任务，遵循先进先出。

> 宏任务：script全部代码、setInterval、setTimeout、setImmediate
>
> 微任务：Process.nexTick、Promise、MutationObserver

工作原理

- 所有任务会被放在调用栈里等待主线程来执行
- js分为同步任务和异步任务，同步任务会按顺序进入调用栈等待主线程依次执行，异步任务会等待到其执行时，将**注册的回调函数**压入任务队列，等待主线程空闲（调用栈被清空），此任务被读取进入调用栈，然后等待被执行。
- 每次主线程空闲时，会去检查微任务队列，如果有微任务，则**一次将微任务队列里的所有微任务**执行完
- 每次主线程空闲、微任务空闲时，会去检查宏任务队列，先执行**一个宏任务**，再检查微任务队列是否有微任务，有则一次执行完微任务队列的微任务，没有则执行下一个宏任务，如此反复，直到宏任务队列被清空。



# 防抖和节流

## 防抖（debounce）

概念：事件被触发在n秒后再执行回调，如果期间事件又被触发，则重新计算时间，n秒后再执行回调

关键步骤：

1. 触发事件
2. setTimeout
3. clearTimeout
4. this指向问题

实现原理：

```js
function debounce(fn, delay) {
    let timer;
    return function () {
        // 清除定时器
        timer && clearTimeout(timer);
        // 重新计算
        timer = setTimeout(() => {
            // 注意这里，改变fn的this，不然会指向全局对象，因为直接调用相当于全局对象调用，记得传参
            fn.apply(this,arguments);
        }, delay);
    }
}

function test(){
    console.log(this)
}

document.getElementById('id').addEventListener('keyup',debounce(test,1000)); //
```

应用场景：用户搜索（用户停止输入后，再去请求）

## 节流 

概念：规定一个单位时间内，只能触发一次函数，这段时间触发多次函数的话，只有一次生效。

实现原理：

```js
function throttle(fn,delay){
    let timer = null;
    return function(){
        // 证明还在执行，直接返回
        if(timer) return;
        fn.apply(this,arguments);
        console.log(new Date())
        timer = setTimeout(()=>{
            //运行完成，置空
            timer = null;
        },delay);
    }
}

function func() {
	console.log(1);
}

document.getElementById('btn').addEventListener('click', throttle(func, 2000));
```

应用场景：滚动事件（频繁触发，限制）



# 0.1+0.2===0.3？

```js
0.1 + 0.2 === 0.3 //false
```

二进制表示，产生无限循环小数

然后将其用科学计数法表示

最后用IEEE754标准的64位双精度表示，**相加时会产生进位**

- 1位符号位，0表示整数
- 11位阶码，表示数字的整数部分，偏移量（1023）+指数位
- 52位尾数，表示数字的小数部分

如果要让0.1+0.2===0.3结果为true呢？

需设置一个误差范围（机器精度），js这个值通常为2^-52^

```js
function numberepsilon(n1,n2){
    return Math.abs(n1-n2) < Number.EPSILON;
}

console.log(numberepsilon(0.1+0.2,0.3)); //true
```



# sort原理

默认采用**字符的unicode**进行排序

传入的比较函数采用以下规则

- 若 a 小于 b，在排序后的数组中 a 应该出现在 b 之前，则返回一个小于 0 的值
- 若 a 等于 b，则返回 0
- 若 a 大于 b，则返回一个大于 0 的值

V8 引擎 sort 函数只给出了两种排序 InsertionSort 和 QuickSort，数量小于10的数组使用 InsertionSort，比10大的数组则使用 QuickSort。

> 现在好像换成冒泡排序了？



# isNaN和Number.isNaN的区别

- isNaN会尝试将这个数转为Number类型，如果不能转则返回true，因此这种方法不是很精确
- Number.isNaN会先判断该数是否为Number类型，然后再进行操作



# Object.is()

和三等号一样，只不过加了些特殊处理，比如NaN等于NaN，+0、-0不再相等



# ajax请求

```js
let url = 'http://locahost:3000';

const xhr = new XMLHttpRequest();

xhr.open('GET',url,true);

xhr.onreadystatechange = function(){
    if(this.readyState !== 4) return;
    if(this.status === 200){
        console.log(this.response);
    } else {
        console.log(this.statusText);
    }
}

xhr.onerror = function(){
    console.log(this.statusText);
}

// 设置响应类型
xhr.responseType = "json";
// 设置请求头
xhr.setRequestHeader("Accept","application/json");

xhr.send(null);
```



# 对象扁平化

```js
function flat(obj){
    const res = {};

    const queue = [];
    queue.push([obj,""]);

    while(queue.length){
        const [node,prix] = queue.shift();
        for(let key in node){
            if(typeof node[key] === "object"){
                queue.push([node[key],prix+key+"."])
            } else {
                res[prix+key] = node[key];
            }
        }
    }

    return res;
}
```

