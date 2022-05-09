# 箭头函数

```js
()=>{
    //...
}
```

- 没有this，捕获外层的this作为本身的this
- 没有原型（prototype）
- 不能使用arguments参数
- 没有new.target

> 所以没办法new



# Promise

介绍：

- promise主要用于异步编程，es6之前通常都是用回调函数来处理异步任务的。

- promise有三个状态**进行中 -- pending、已成功 -- fullfilled（resolved）、已失败 -- rejected**
  - 只能`pending -> fullfilled`或者`pending -> rejected`
  - 状态改变后不会再改变，且状态不可逆
- 如果要实例化一个promise对象，就给Promise类传进一个函数，函数参数为resolve和reject两个方法
  - 执行**resolve**就会从**pending状态**进入**resolved状态**，会得到一个成功的值（value），异步操作成功时调用。
  - 执行**reject**就会从**pending状态**进入**rejected状态**，会得到一个失败原因（reason），异步操作失败时调用。
  - then方法接收两个参数，参数都是函数，第一个参数是用来处理之前resolve抛出的结果，第二个参数是用来处理reject抛出的结果。
  - 内部抛出错误，就可以catch去捕获这个错误

实现原理：

```js
class MyPromise {
    static pending = 'pending';
    static fulfilled = 'fulfilled';
    static rejected = 'rejected';

    constructor(executor) {
        // 状态
        this.state = MyPromise.pending;
        // 成功的值
        this.value = undefined;
        // 失败原因
        this.reason = undefined;
        // 存放成功的数组
        this.resolveQueue = [];
        // 存放失败的数组
        this.rejectQueue = [];

        let resolve = (value) => {
            // 保证状态不可逆
            if (this.state === MyPromise.pending) {
                this.state = MyPromise.fulfilled;
                this.value = value;
                this.resolveQueue.forEach(fn => fn());
            }
        }

        let reject = (reason) => {
            // 保证状态不可逆
            if (this.state === MyPromise.pending) {
                this.state = MyPromise.rejected;
                this.reason = reason;
                this.rejectQueue.forEach(fn => fn());
            }
        }

        // 执行出错直接进入reject
        try {
            executor(resolve, reject);
        } catch (e) {
            reject(e);
        }
    }

    then(onFulfilled, onRejected) {
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
        onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err };

        let promise2 = new MyPromise((resolve, reject) => {
            if (this.state === MyPromise.fulfilled) {
                setTimeout(() => {
                    let x = onFulfilled(this.value);
                    resolvePromise(promise2, x, resolve, reject);
                }, 0)
            }

            if (this.state === MyPromise.rejected) {
                setTimeout(() => {
                    let x = onRejected(this.reason);
                    resolvePromise(promise2, x, resolve, reject);
                }, 0)
            }

            // 如果为pending状态，则存放到数组中
            if (this.state === MyPromise.pending) {
                this.resolveQueue.push(() => {
                    setTimeout(() => {
                        let x = onFulfilled(this.value);
                        resolvePromise(promise2, x, resolve, reject);
                    }, 0)
                });
                this.rejectQueue.push(() => {
                    setTimeout(() => {
                        let x = onRejected(this.reason);
                        resolvePromise(promise2, x, resolve, reject);
                    }, 0)
                });
            }
        })

        return promise2;
    }

    catch(onRejected) {
	
    }
    
    static resolve(val) {
        return new MyPromise((resolve) => {
            resolve(val);
        })
    }

    static reject(reason) {
        return new MyPromise((resolve, reject) => {
            reject(reason);
        })
    }

    static race(promises) {
        return new MyPromise((resolve, reject) => {
            for (let i = 0; i < promises.length; i++) {
                promises[i].then(resolve, reject);
            } 
        })
    }

    static all(promises) {
        let arr = [];
        let i = 0;
        function process(index,data){
            arr[index] =data;
            i++;
            if(i === promises.length){
                resolve(arr);
            }
        }

        return new MyPromise((resolve,reject)=>{
            for(let i = 0; i < promises.length; i++){
                promise[i].then(data=>{
                    process(i,data);
                },reject);
            }
        })
    }
}


function resolvePromise(promise, x, resolve, reject) {
    if (promise === x)
        return reject(new TypeError('Chaining cycle detected for promise #<Promise>'));
    if (typeof x === 'object' || typeof x === 'function') {
        if (x === null) {
            return resolve(x);
        }

        let then;

        try {
            then = x.then;
        } catch (err) {
            return reject(err);
        }

        if (typeof then === 'function') {
            // 防止多次调用
            let called = false;

            try {
                then.call(
                    x,
                    y => {
                        if (called) return;
                        called = true;
                        resolvePromise(promise, y, resolve, reject);
                    },
                    r => {
                        if (called) return;
                        called = true;
                        reject(r);
                    }
                )
            } catch (e) {
                if (called) return;
                reject(e);
            }
        } else {
            // 如果 then 不是函数，以 x 为参数执行 promise
            resolve(x);
        }
    } else {
        // 如果 x 不为对象或者函数，以 x 为参数执行 promise
        resolve(x);
    }

}

// 简单实现
// function resolvePromise(promise2, x, resolve, reject) {
//     if (promise2 === x) return reject(new TypeError('Chaining cycle detected for promise #<Promise>'));
//     if (x instanceof MyPromise) {
//         x.then(resolve, reject);
//     } else {
//         resolve(x);
//     }
// }

```

## Promise.all

Promise.all会将多个Promise实例包装，返回一个新的promise实例，成功会返回一个结果数组，与原来的promise一一映射，失败的话返回最先失败的那个结果

```js


Promise.all = function (promises) {
    let count = 0;
    let arr = new Array(promises.length);

    return new Promise((resolve, reject) => {
        for (let i = 0; i < promises.length; i++) {
            if (promises[i] instanceof Promise) {
                promises[i].then(res => {
                    arr[i] = res;
                    count++;

                    if (count === promises.length) {
                        resolve(arr);
                    }
                }).catch(err=>{
                    reject(err);
                })
            } else {
                arr[i] = promises[i];
                count++;
                if (count === promises.length) {
                    resolve(res);
                }
            }

        }
    })
}

let a = new Promise((resolve, reject) => { setTimeout(()=>{
    resolve('timeout');
})});
let b = 137481;
let c = Promise.resolve(141);
let d = Promise.reject("err");

Promise.all([a, b, c,d]).then(res => { console.log(res) })
```



## Promise.race

Promise.race将多个Promise实例包装起来，返回一个新的Promise实例，哪个结果计算的快就返回它



# Iterator（遍历器）

iterator是一种接口，给各种不同数据结构提供统一的访问机制，部署了Iterator接口，就可以采用`for...of`遍历。**Symbol.iterator**

工作原理：

1. 创建一个指针对象，指向当前数据结构的初始位置
2. 第一次调用next指针对象的next对象，指向数据结构的第一个成员
3. 不断调用next，直到指向数据结构的结束位置
4. 每次调用next，返回的是一个带有value、done属性的对象

```js
const arr = [1,2,4,5];
let iterator = arr[Symbol.iterator]();

console.log(iterator.next()); //{value: 1, done: false}
console.log(iterator.next()); //{value: 2, done: false}
console.log(iterator.next()); //{value: 4, done: false}
console.log(iterator.next()); //{value: 5, done: false}
console.log(iterator.next()); //{value: undefined, done: true}
```

当前有iterator接口的数据结构

- Array
- Map
- Set
- arguments以及类似数组的对象
- string

实现原理：

```js
//给对象添加可遍历for。。。of方法

let obj = {
    a: 1,
    b: 2,
    c: 3,
}

Object.prototype[Symbol.iterator] = function () {
    //this指向调用该方法的对象
    const keys = Object.keys(this);
    let count = 0;
    let that = this;

    return {
        next: function () {
            if (count < keys.length) {
                return { value: that[keys[count++]], done: false }
            }
            return { value: undefined, done: true }
        }
    }
}

for (const v of obj){
    console.log(v);
}

//1 2 3
```

总结：iterator是个对象，有个next方法



# generator(生成器)

异步编程，简单理解，其实就是一个状态机，封装了多个内部状态，执行generator函数会返回一个遍历器对象。

简单使用：

- function和函数名中间带个*，即`function* func()`
- 使用`yield`，定义不同的内部状态
- 调用next，会在遇到yield语句就停止执行，并将yield语句后的值作为value返回（最后的value值是return的值），done的话直到return或者没有yield时值为true时
- 注意在函数外第二次调用next，并传入参数，该参数会作为第一次yield返回的结果。以此类推

```js
function* gen(arg) {
    console.log(arg)
    let one = yield '1';
    console.log(one);
    let two = yield '2';
    console.log(two);
    return 'nihao';
}

let p = gen('AAA');
console.log(p);
console.log(p.next());
console.log(p.next('BBB'));
console.log(p.next('CCC'));
```

实现原理：**保存其函数的执行上下文**，每次next，其实都执行了一遍传入的生成器函数，只是中间存储了上下文，使得可以从上一次的执行结果开始执行。使用context对象保存上下文。



# async/await

genrator的语法糖，使其可以自动执行，不需手动调用next

async函数

- 返回的结果是成功或失败的**Promise类型的对象**，
- 如果内部抛出错误则为失败的Promise，否则为成功的Promise。

await命令

- 正常情况下后面是一个Promise对象，返回该对象成功执行的结果
- 如果后面不是Promise对象，则直接返回后面的值
- await出错了，可以用`try...catch`捕获

> async函数里，遇到await时，执行完await紧跟的语句，会先跳出函数，等执行完异步操作后，再执行原来函数位置下面的语句。相当于会产生一个微任务（下面语句）

async实现原理：

- 简单实现，yield后面跟Promise

```js
function run(gen) {
    const p = gen();

    function next(data) {
        const res = p.next(data);
        if (res.done) return res.value;
        res.value.then(data => {
            next(data);
        })
    }

    next();
}

function* gen(){
    const one = yield (()=>new Promise((resolve,reject)=>resolve(1)))();
    console.log(one);
    const two = yield (()=>new Promise((resolve,reject)=>resolve(2)))();
    console.log(two);i 
    return 3;
}

run(gen);
```

- 真正实现async，增加了Promise，容错处理

```js
function asyncGen(gen){
    return new Promise((resolve,reject)=>{
        const g = gen();
        
        function step(nextF){
            let next;
            try(
            	next = nextF();
            ) catch(e){
                reject(e);
            }
            
            if(next.done) return resolve(next.value);
            
            Promise.resolve(next.value).then(
            	res=>step(function(){return g.next(next.value)}),
                err=>step(function(){g.throw(err)})
            )
        }
        
        step(function(){return g.next()});
    })
}

function* genTest(){
    let one = yield 1;
    console.log(one);
    let two = yield 2;
    console.log(two);
    return 3;
}

asyncGen(genTest); //1 2 3

//另一种写法
function CO(gen){
    const g = gen();
    
    return new Promise((resolve,reject)=>{
        let next = (data)=>{
            let {value,done} = g.next(data);
            if(done){
                resolve(value);
            } else {
                value instanceof Promise?value.then(res=>{
                    next(res)
                },):next(value);
            }
        }
        next();
    })
}
```



# 模块化

将一个大的文件，拆分为小的文件，再将这些小文件组合起来

优点：代码复用，防止命名冲突，高维护性

模块化规范：

- commonjs：nodejs，browserify。同步加载
- amd：requirejs。异步模块定义，在浏览器端使用，异步加载，依赖前置。
- cmd：seajs。通用模块定义，异步加载，就近加载。
- es6 module

>es6是编译时加载模块

## es6 module和commonjs模块

- es6 module
  - **es6 module是对模块的引用**，动态只读，类似const，不能改变指针的指向，但是可以改变内部值
  - 编译时输出接口，执行时根据接口去引用模块
- commonjs模块
  - **commonjs是对模块的浅拷贝**，都可以改变本身和内部指针指向
  - 执行时才去加载模块

## babel

js编译器，将一些新的语法转换为浏览器能识别的语法



# Map

## map和object

- 键不同，map的键可以任意类型，object只能使用字符串
- 遍历顺序不同，map按插入顺序遍历，
  - object遍历顺序
    - 首先遍历所有数字键，按照数值升序排列
    - 其次遍历所有字符串键，按照加入时间升序排列
    - 最后遍历Symbol键，按照加入时间升序排列

## map和WeakMap

- map的键可以是任意类型的值
- weakmap的键只能是对象，是弱引用，一旦weakmap里面的对象没有被引用，即会被垃圾回收机制给清除掉



# Proxy（代理）

proxy用于修改某些操作的默认行为，在目标对象之前设一层拦截，外界对该对象的访问，都必须进行这层拦截，对外界的访问进行过滤和改写。

Proxy类没有显式原型

```js
Proxy.prototype; //undefined
Proxy.__proto__ === Function.prototype; //true

const person = function(name){this.name = name};

const proxy = new Proxy(person,{});

person.__proto__ === proxy.__proto__; //true

typeof p

const a = new proxy('xhw');
a.__proto__ === proxy.prototype; // true
a.__proto__ === person.prototype; //true
```

**应用**：

```js
let onWatch = (obj, setBind, getLogger) => {
    let handler = {
        get(target, property, receiver) {
            getLogger(target, property);
            return Reflect.get(target, property, receiver)
        },

        set(target, property,value,receiver) {
            setBind(value,property);
            return Reflect.set(target, property, value, receiver);
        }
    }

    return new Proxy(obj, handler);
}

let obj = { a: 1 };

let p = onWatch(obj,
    (v,p)=>{
        console.log(`${p}的值改为${v}`);
    },
    (v,p)=>{
        console.log(`${p}的值为${v[p]}`);
    }
)

p.a = 2;
p.a;
```



# Reflect（反射）

操作对象的api

- 将`Object`对象的一些属于语言内部的方法（`Object.defineProperty`）放到`Reflect`对象上等等操作

```js
Reflect.has(obj,'prototype'); //true
```

 

# class类

一个语法糖，让对象原型的写法更简洁

## 关于new.target

函数通过new调用，new.target就指向此函数，否则指向undefined

```js
function F(){
    if(!new.target) throw '不给你执行';
    console.log('exe');
}

F(); //报错
new F()
```



# 继承

结合js的继承来比较

es6的继承是先创建个空对象，将父类的属性、方法复制给其，再将该对象作为子类实例。所谓“继承在前，实例在后”。先调用super这一步会返回个继承父类的this对象。

不同说法：在父类中创建this，之后将this传给子类，子类再对其进行修改

**源码**：先继承父类的静态方法、属性以及父类原型的方法、属性，再创建子类原型的方法、属性，new时再去继承父类的属性、方法，创建子类的属性、方法。

es5的继承是先创建子类实例，再将父类属性、方法复制给其。“实例在前，继承在后”

> 所以新建子类实例时，会先调用父类的构造函数（super()）

**注意：**

super的角色

- 作为函数时，只能在constructor中使用
- 作为对象时
  - 在普通函数中，指向父类的原型对象
  - 在静态方法中，指向父类

```js
class Person{
    constructor(){
        this.name = 'test';
    }
}

//相当于

class Person {
    name = 'test';
    constructor(){
        
    }
}
```



es6转码

```js
class A {
    constructor(name) {
        this.name = name;
    }

    sayName(){
        console.log(this.name);
    }
}

class B extends A {

    constructor(name,age) {
        super(name);
        this.age = age;
    }

    sayAge() {
        console.log('say');
    }
}

console.log(new B('b',12));
```



```js
'use strict';

var _createClass = function () {
    function defineProperties(target, props) {
        for (var i = 0; i < props.length; i++) {
            var descriptor = props[i];
            descriptor.enumerable = descriptor.enumerable || false;
            descriptor.configurable = true;
            if ("value" in descriptor) descriptor.writable = true;
            Object.defineProperty(target, descriptor.key, descriptor);
        }
    } 
    return function (Constructor, protoProps, staticProps) {
        if (protoProps) defineProperties(Constructor.prototype, protoProps);
        if (staticProps) defineProperties(Constructor, staticProps);
        return Constructor;
    };
}();

function _possibleConstructorReturn(self, call) {
    if (!self) {
        throw new ReferenceError("this hasn't been initialised - super() hasn't been called");
    }
    return call && (typeof call === "object" || typeof call === "function") ? call : self;
}

//完成父类静态方法、属性的继承以及原型方法、属性的继承
function _inherits(subClass, superClass) {
    if (typeof superClass !== "function" && superClass !== null) {
        throw new TypeError("Super expression must either be null or a function, not " + typeof superClass);
    }
    subClass.prototype = Object.create(superClass && superClass.prototype, {
        constructor: {
            value: subClass,
            enumerable: false,
            writable: true,
            configurable: true
        }
    });
    if (superClass) Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;
}

function _classCallCheck(instance, Constructor) {
    if (!(instance instanceof Constructor)) {
        throw new TypeError("Cannot call a class as a function");
    }
}

var A = function () {
    function A(name) {
        _classCallCheck(this, A);

        this.name = name;
    }

    _createClass(A, [{
        key: 'sayName',
        value: function sayName() {
            console.log(this.name);
        }
    }]);

    return A;
}();

var B = function (_A) {
    _inherits(B, _A);

    function B(name, age) {
        _classCallCheck(this, B);

        var _this = _possibleConstructorReturn(this, (B.__proto__ || Object.getPrototypeOf(B)).call(this, name));

        _this.age = age;
        return _this;
    }

    _createClass(B, [{
        key: 'sayAge',
        value: function sayAge() {
            console.log('say');
        }
    }]);

    return B;
}(A);

console.log(new B('b', 12));
```



## 关于class的静态方法

至于为什么子类还可以继承父类的静态方法，这说明class是有两个继承链的。

- 子类的\_\_proto\_\_表示构造函数的继承，指向父类
- 子类prototype属性的\_proto\_属性表示方法的继承，指向父类的原型

```js
class A{
    
}

class B extends A{
    
}

B.prototype.__proto__ === A.prototype; //B的原型的原型对象指向A的原型对象，此时B.prototype其实是A的实例
B.__proro_ === A; //B的构造函数的原型指向A的构造函数
```



# 新特性

## BigInt

number类型只能表示-(2^53^-1)~ 2^53^ -1间的整数，BigInt可以表示更大的数。尾数+个**n**表示BigInt类型，可用二进制、八进制、十进制、十六进制表示前面的数字

```js
BigInt(1) === 1n //true
```

BigInt类型可以转换为字符串

```js
BigInt(1).toString(); //1
```

## 可选链

访问多层级对象，不需要进行冗余的前置检验

```js
//之前
var name = user && user.info && user.info.name;

//现在
var name = user?.info?.name;
```



# ArrayBuffer

二进制数组，设计初衷是为了WebGL（浏览器与显卡之间的通信接口），方便js与显卡之间进行数据交换，通信使用的是二进制数据。所以ArrayBuffer的使用可以提升性能。

ArrayBuffer对象代表原始的二进制数据，不能直接读写。

读写二进制数据的话，需要创建视图来对ArrayBuffer进行操作

- TypedArray通过下标访问法修改和读取视图
- DataView提供get的API对ArrayBuffer进行访问，提供set的API对视图进行修改。

