## 防抖和节流
### 防抖 debounce
* 当一次事件发生后，事件处理器要等一定阈值的时间，如果这段时间过去后 再也没有 事件发生，就处理最后一次发生的事件。假设还差 0.01 秒就到达指定时间，这时又来了一个事件，那么之前的等待作废，需要重新再等待指定时间。
* 使用场景：
1. search搜索联想，用户在不断输入值时，用防抖来节约请求资源。
2. window触发resize的时候，不断的调整浏览器窗口大小会不断的触发这个事件，用防抖来让其只触发一次 
* 代码 [参考](https://github.com/mqyqingfeng/Blog/issues/22)
```
function debounce(fn, wait=50, immediate=false){
    let timer;
    return function() {
        var context = this;
        if(immediate) {
            fn.apply(this, arguments)
        }
        if(timer) {
            clearTimeout(timer)
        }
        timer = setTimeout(function(){
            fn.apply(context, arguments)
        }, wait)
    }
}
```
### 节流 throttle
* 事件在一个管道中传输，加上这个节流阀以后，事件的流速就会减慢。实际上这个函数的作用就是如此，它可以将一个函数的调用频率限制在一定阈值内，例如 1s，那么 1s 内这个函数一定不会被调用两次
* 使用场景：
1. 比如在滚动的时候要检查当前滚动的位置，来显示或隐藏回到顶部按钮
* 时间戳实现方式
```
function throttle(func, wait) {
    let prev = new Date();;
    return function() {
        let context = this;
        let now = new Date();
        if (now - prev > wait) {
            func.apply(context, arguments);
            prev = now;
        }
    }
}
```
* 定时器实现方式
```
function throttle(func, wait) {
    var timeout;
    return function() {
        let context = this;
        if (!timeout) {
            timeout = setTimeout(function(){
                timeout = null;
                func.apply(context, arguments)
            }, wait)
        }

    }
}
```
* 比较

第一种事件会立刻执行，第二种事件会在 n 秒后第一次执行
第一种事件停止触发后没有办法再执行事件，第二种事件停止触发后依然会再执行一次事件
* 结合版 [参考](https://juejin.im/post/5be24d76e51d451def13cca2)
```
function throttle(func, wait) {
    var timeout, context, args, result;
    var previous = 0;
    var later = function() {
        previous = +new Date();
        timeout = null;
        func.apply(context, args)
    };
    var throttled = function() {
        var now = +new Date();
        //下次触发 func 剩余的时间
        var remaining = wait - (now - previous);
        context = this;
        args = arguments;
         // 如果没有剩余的时间了或者你改了系统时间
        if (remaining <= 0 || remaining > wait) {
            if (timeout) {
                clearTimeout(timeout);
                timeout = null;
            }
            previous = now;
            func.apply(context, args);
        } else if (!timeout) {
            timeout = setTimeout(later, remaining);
        }
    };
    return throttled;
}
```

## promise
* Promise 是一个对象，它代表了一个异步操作的最终完成或者失败
* 在本轮 事件循环 运行完成之前，回调函数是不会被调用的。
* 即使异步操作已经完成（成功或失败），在这之后通过 then() 添加的回调函数也会被调用。
* 通过多次调用 then() 可以添加多个回调函数，它们会按照插入顺序执行。
* 链式调用
* Promise 被拒绝时，会有下文所述的两个事件之一被派发到全局作用域（通常而言，就是window）
    * rejectionhandled
    当 Promise 被拒绝、并且在 reject 函数处理该 rejection 之后会派发此事件。
    * unhandledrejection
    当 Promise 被拒绝，但没有提供 reject 函数来处理该 rejection 时，会派发此事件。
    
    以上两种情况中，PromiseRejectionEvent 事件都有两个属性，一个是 promise 属性，该属性指向被驳回的 Promise，另一个是 reason 属性，该属性用来说明 Promise 被驳回的原因。
* Promise.all()和Promise.race：接受一个promise对象组成的数组作为参数。
    * all--每个子状态都变成fulfilled，最终状态才会fulfilled，返回值以数组给then；只要有一个reject，则最终状态reject，返回第一个被reject的实例给catch。
    * race--返回第一个改变状态的状态和返回值，不是数组。
* 手写promise [例子](https://juejin.im/post/5dc383bdf265da4d2d1f6b23#heading-8)
```
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";
function Promise(excutor) {
    let that = this; // 缓存当前promise实例对象
    that.status = PENDING; // 初始状态
    that.value = undefined; // fulfilled状态时 返回的信息
    that.reason = undefined; // rejected状态时 拒绝的原因
    that.onFulfilledCallbacks = []; // 存储fulfilled状态对应的onFulfilled函数
    that.onRejectedCallbacks = []; // 存储rejected状态对应的onRejected函数
    function resolve(value) { // value成功态时接收的终值
        if(value instanceof Promise) {
            return value.then(resolve, reject);
        }
        // 实践中要确保 onFulfilled 和 onRejected 方法异步执行，且应该在 then 方法被调用的那一轮事件循环之后的新执行栈中执行。
        setTimeout(() => {
            // 调用resolve 回调对应onFulfilled函数
            if (that.status === PENDING) {
                // 只能由pending状态 => fulfilled状态 (避免调用多次resolve reject)
                that.status = FULFILLED;
                that.value = value;
                that.onFulfilledCallbacks.forEach(cb => cb(that.value));
            }
        });
    }
    function reject(reason) { // reason失败态时接收的拒因
        setTimeout(() => {
            // 调用reject 回调对应onRejected函数
            if (that.status === PENDING) {
                // 只能由pending状态 => rejected状态 (避免调用多次resolve reject)
                that.status = REJECTED;
                that.reason = reason;
                that.onRejectedCallbacks.forEach(cb => cb(that.reason));
            }
        });
    }
    // 捕获在excutor执行器中抛出的异常
    // new Promise((resolve, reject) => {
    //     throw new Error('error in excutor')
    // })
    try {
        excutor(resolve, reject);
    } catch (e) {
        reject(e);
    }
}
Promise.prototype.then = function(onFulfilled, onRejected) {
    const that = this;
    let newPromise;
    // 处理参数默认值 保证参数后续能够继续执行
    onFulfilled =
        typeof onFulfilled === "function" ? onFulfilled : value => value;
    onRejected =
        typeof onRejected === "function" ? onRejected : reason => {
            throw reason;
        };
    if (that.status === FULFILLED) { // 成功态
        return newPromise = new Promise((resolve, reject) => {
            setTimeout(() => {
                try{
                    let x = onFulfilled(that.value);
                    resolvePromise(newPromise, x, resolve, reject); // 新的promise resolve 上一个onFulfilled的返回值
                } catch(e) {
                    reject(e); // 捕获前面onFulfilled中抛出的异常 then(onFulfilled, onRejected);
                }
            });
        })
    }
    if (that.status === REJECTED) { // 失败态
        return newPromise = new Promise((resolve, reject) => {
            setTimeout(() => {
                try {
                    let x = onRejected(that.reason);
                    resolvePromise(newPromise, x, resolve, reject);
                } catch(e) {
                    reject(e);
                }
            });
        });
    }
    if (that.status === PENDING) { // 等待态
        // 当异步调用resolve/rejected时 将onFulfilled/onRejected收集暂存到集合中
        return newPromise = new Promise((resolve, reject) => {
            that.onFulfilledCallbacks.push((value) => {
                try {
                    let x = onFulfilled(value);
                    resolvePromise(newPromise, x, resolve, reject);
                } catch(e) {
                    reject(e);
                }
            });
            that.onRejectedCallbacks.push((reason) => {
                try {
                    let x = onRejected(reason);
                    resolvePromise(newPromise, x, resolve, reject);
                } catch(e) {
                    reject(e);
                }
            });
        });
    }
};
```
## 计时器
* 使用requestAnimationFrame实现定时器
```
class RAF {
    constructor () {
        this.init()
    }
    init () {
        this._timerIdMap = {
            timeout: {},
            interval: {}
        }
    }
    run (type = 'interval', cb, interval = 16.7) {
        const now = Date.now
        let stime = now()
        let etime = stime
        // 创建Symbol类型作为key值，保证返回值的唯一性，用于清除定时器使用
        const timerSymbol = Symbol(String)
        const loop = () => {
            this.setIdMap(timerSymbol, type, loop)
            etime = now()
            if (etime - stime >= interval) {
                if (type === 'interval') {
                    stime = now()
                    etime = stime
                }
                cb()
                type === 'timeout' && this.clearTimeout(timerSymbol)
            }
        }
        this.setIdMap(timerSymbol, type, loop)
        return timerSymbol // 返回Symbol保证每次调用setTimeout/setInterval返回值的唯一性
    }
    setIdMap (timerSymbol, type, loop) {
        const id = requestAnimationFrame(loop)
        this._timerIdMap[type][timerSymbol] = id
    }
    setTimeout (cb, interval) { // 实现setTimeout 功能
        return this.run('timeout', cb, interval)
    }
    clearTimeout (timer) {
        cancelAnimationFrame(this._timerIdMap.timeout[timer])
    }
    setInterval (cb, interval) { // 实现setInterval功能
        return this.run('interval', cb, interval)
    }
    clearInterval (timer) {
        cancelAnimationFrame(this._timerIdMap.interval[timer])
    }
}
export default RAF
```
## eventEmitter
* 实现简易版eventEmitter
```
    class EventEmitter {
        constructor () {
            this.handlers={};
        }
        on(eventName, callback){
            if(!this.handlers[eventName]){
                this.handlers[eventName]=[];
            }
            this.handlers[eventName].push(callback);
        }
        emit(eventName, ...arg){
            if(this.handlers[eventName]){
                for(var i=0;i<this.handlers[eventName].length;i++){
                    this.handlers[eventName][i](...arg);
                }
            }
        }
        off(eventName, handle) {
                if (!this.handlers.hasOwnProperty(eventName)) return
            //获取下标，并删除
            let index = this.handlers[eventName].indexOf(handle)
            this.handlers[eventName].splice(index, 1)
        }
        once(eventName, callback){
            let wrapFanc = (...args) => {
                callback.apply(this.args)
                this.off(eventName,wrapFanc)
            }
            this.on(eventName,wrapFanc)
        }
    }
    function a () {
        console.log(1111)
    }
    let event = new EventEmitter();
    event.on('say',function(str){
        console.log(str);
    });
    event.once('say', ()=>{
        console.log('once')
    });
    event.emit('say', 'hello world');
    event.emit('say', 'hello world');
    event.on('event', a);
    event.on('event', ()=>{
        console.log(2222)
    });
    event.off('event', a);
    event.emit('event');
```
## 拷贝
### 数组的浅拷贝
* slice、concat 返回一个新数组的特性来实现拷贝
```
var arr = ['old', 1, true, null, undefined];
var new_arr = arr.concat();
var new_arr = arr.slice();
new_arr[0] = 'new';
console.log(arr) // ["old", 1, true, null, undefined]
console.log(new_arr) // ["new", 1, true, null, undefined]
```
### 手写一个深拷贝
```
function deepCopy(obj) {
    if(typeof obj === 'object') {
        //复杂数据类型
        var result = Array.isArray(obj) ? [] : {};
        for(let i in obj){
            result[i] = typeof obj[i] == "object" ? deepCopy(obj[i]) : obj[i];
        }  
    }else {
        var result = obj;
    }
}
```
### 实现一个instanceof
```
function instanceOf(left,right) {
    let proto = left.__proto__;
    let prototype = right.prototype
    while(true) {
        if(proto === null) return false
        if(proto === prototype) return true
        proto = proto.__proto__;
    }
}
```

## new操作符
### 步骤
1. 创建一个新对象
2. 将构造函数的作用域赋给新对象，this指向该对象
3. 执行构造函数中的代码，为新对象添加属性
4. 返回新对象

### 代码实现new函数
```
function New(func) {
    var res = {};
    if (func.prototype !== null) {
        res.__proto__ = func.prototype;
    }
    // Array.prototype.slice.call(arguments, 1)获取除第一个参数除外其他参数
    var ret = func.apply(res, Array.prototype.slice.call(arguments, 1));
    if ((typeof ret === "object" || typeof ret === "function") && ret !== null) {
        return ret;
    }
    return res;
}
var obj = New(A, 1, 2);
// equals to
var obj = new A(1, 2);
```

## 继承
### 代码实现
```
function Parent(name) {
    this.name = name;
}
Parent.prototype.sayName = function() {
    console.log('parent name:', this.name);
}
function Child(name, parentName) {
    Parent.call(this, parentName);  
    this.name = name;    
}
function inheritPrototype(subType, superType) {
    var prototype = object(superType.prototype);
    prototype.constructor = subType;
    subType.prototype = prototype;
}
function create(proto) {
    function F(){}
    F.prototype = proto;
    return new F();
}
// 1.Child.prototype = create(Parent.prototype);
// Child.prototype = new Parent(); 导致多走一次parent
// 2.
Child.prototype = inheritPrototype(Child, Parent);
Child.prototype.sayName = function() {
    console.log('child name:', this.name);
}
Child.prototype.constructor = Child;
var parent = new Parent('father');
parent.sayName();    // parent name: father
var child = new Child('son', 'father');
```

## 函数柯里化
* 运行过程其实是一个参数的收集过程，我们将每一次传入的参数收集起来，并在最里层里面处理。

### 简易版
```
function curry(fn, args) {
    var length = fn.length;
    var args = args || [];
    return function () {
        newArgs = args.concat(Array.prototype.slice.call(arguments));
        if(newArgs.length < length) {
            return curry.call(this, fn, newArgs)
        } else {
            return fn.apply(this, newArgs)
        }
    }
}
function multiFn(a, b, c) {
    return a * b * c;
}
var multi = curry(multiFn);
multi(2)(3)(4);
multi(2,3,4);
multi(2)(3,4);
multi(2,3)(4);
```

### ES6
```
const curry = (fn, arr = []) => (...args) => (
  arg => arg.length === fn.length
    ? fn(...arg)
    : curry(fn, arg)
)([...arr, ...args])

let curryTest=curry((a,b,c,d)=>a+b+c+d)
curryTest(1,2,3)(4) //返回10
curryTest(1,2)(4)(3) //返回10
curryTest(1,2)(3,4) //返回10
```

## bind,call,apply
### 模拟apply
[参考](https://github.com/mqyqingfeng/Blog/issues/11)
* 思路 
```
var foo = {
    value: 1,
    bar: function() {
        console.log(this.value)
    }
};
foo.bar();
```
* 步骤
    1. 函数设为对象的属性
    2. 执行该函数
    3. 删除该函数

* 代码

call函数
```
Function.prototype.call2 = function(content = window) {
    content.fn = this;
    let args = [...arguments].slice(1);
    let result = content.fn(...args);
    delete content.fn;
    return result;
}
let foo = {
    value: 1
}
function bar(name, age) {
    console.log(name)
    console.log(age)
    console.log(this.value);
}
bar.call2(foo, 'black', '18') // black 18 1
```

apply函数
```
Function.prototype.apply2 = function(context = window) {
    context.fn = this
    let result;
    // 判断是否有第二个参数
    if(arguments[1]) {
        result = context.fn(...arguments[1])
    } else {
        result = context.fn()
    }
    delete context.fn
    return result
}
```

### 实现一个bind
[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
```
Function.prototype.bind = function() {
    var slice = Array.prototype.slice;
    var thatFunc = this, thatArg = arguments[0];
    var args = slice.call(arguments, 1);
    if (typeof thatFunc !== 'function') {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - ' +
             'what is trying to be bound is not callable');
    }
    return function(){
      var funcArgs = args.concat(slice.call(arguments))
      return thatFunc.apply(thatArg, funcArgs);
    };
};
```

## const let var
### const
```
    var __const = function __const (data, value) {
        window.data = value // 把要定义的data挂载到window下，并赋值value
        Object.defineProperty(window, data, { // 利用Object.defineProperty的能力劫持当前对象，并修改其属性描述符
          enumerable: false,
          configurable: false,
          get: function () {
            return value
          },
          set: function (data) {
            if (data !== value) { // 当要对当前属性进行赋值时，则抛出错误！
              throw new TypeError('Assignment to constant variable.')
            } else {
              return value
            }
          }
        })
      }
      __const('a', 10)
      console.log(a)
      delete a
      console.log(a)
      for (let item in window) { // 因为const定义的属性在global下也是不存在的，所以用到了enumerable: false来模拟这一功能
        if (item === 'a') { // 因为不可枚举，所以不执行
          console.log(window[item])
        }
      }
    a = 20 // 报错
```
### var
var变量声明提升, const和let却不会

### const let 
* 临时死区
```
if(condition) {
    console.log(typeof value);
    // VM107:2 Uncaught ReferenceError: Cannot access 'value' before initialization
    let value = "blue"
}
```

    这样却可以
```
// undefined
console.log(typeof value);
if(true) {
    let value = "blue"
}
```
* 用let || const 不能覆盖全局对象，只能遮蔽他（会在全局作用域下创建一个新的绑定，但该绑定不会添加为全局对象的属性）

## Iterator && Generator
* for of 使用于可迭代对象，用于不可迭代对象，如null、undefined会报错
* 数组，map,set, 字符串，nodelist
* 展开运算符可作用于可迭代对象，并转换成数组 [...set]

### 基础
#### es5的迭代器
```
function createIterator(items){
    var i = 0;
    return {
        next: function() {
            var done = (i >= items.length);
            var value = !done ? items[i++] : undefined;
            return {
                done,
                value
            };
        }
    }
}
var iterators = createIterator([1, 2, 3]);
console.log(iterators.next()) // {done: false, value: 1}
console.log(iterators.next()) // {done: false, value: 2}
console.log(iterators.next()) // {done: false, value: 3}
console.log(iterators.next()) // {done: true, value: undefined}
```
#### es6
```
function* makeRangeIterator(start = 0, end = Infinity, step = 1) {
    for (let i = start; i < end; i += step) {
        yield i;
    }
}
var a = makeRangeIterator(1,10,2)
a.next() // {value: 1, done: false}
a.next() // {value: 3, done: false}
a.next() // {value: 5, done: false}
a.next() // {value: 7, done: false}
a.next() // {value: 9, done: false}
a.next() // {value: undefined, done: true}
```
#### 检测对象是否为可迭代对象
对象默认的迭代器 属性为Symbol.iterator的方法
```
function isIterable(obj) {
    return typeof obj[Symbol.iterator] === 'function'
}
```
#### 创建可迭代对象
```
var myIterable = {
  *[Symbol.iterator]() {
    yield 1;
    yield 2;
    yield 3;
  }
}
for (let value of myIterable) { 
    console.log(value); 
}
```
#### 集合对象迭代器
    * entries() 返回一个迭代器，其值为多个键值对
    * values()
    * keys()

### 高级迭代器
    * The next() 方法也接受一个参数用于修改生成器内部状态。传递给 next() 的参数值会被yield接收。
    * 要注意的是，传给第一个 next() 的值会被忽略。
#### 传递参数
```
function* createIterator() {
    let first = yield 'welcome';
    let second = yield first + 'world';
    yield second + 'hard';
}
// next方法不带参数
let iterator = createIterator();
console.log(iterator.next());// { value: 'welcome', done: false }
console.log(iterator.next());// { value: 'undefinedworld', done: false }
console.log(iterator.next()); // { value: 'undefinedhard', done: false }
console.log(iterator.next());//{ value: undefined, done: true }
//next方法带参数
iterator = createIterator();
console.log(iterator.next());//{ value: 'welcome', done: false }
console.log(iterator.next('hello')); //{ value: 'helloworld', done: false }
console.log(iterator.next('work')); //{ value: 'workhard', done: false }
console.log(iterator.next());//{ value: undefined, done: true }
```
#### 抛出错误
* throw()命令迭代器继续执行，但同时抛出一个错误
```
function* createIterator() {
    let first = yield 1;
    let second;
    try {
        second = yield first + 2;
    } catch (ex) {
        second = 6;
    }
    yield second + 3;
}
let iterator = createIterator();
console.log(iterator.next()); //{ value: 1, done: false }
console.log(iterator.next(4)); //{ value: 6, done: false }
console.log(iterator.throw(new Error("Boom"))); //{ value: 9, done: false }
console.log(iterator.next()); //{ value: undefined, done: true }
```

#### 返回语句return
* return 表示所有操作已完成，done被置为true
```
function *createIterator(){
    yield 1;
    return 2;
    yield 3;
    yield 4;
}
let i = createIterator();
console.log(i.next()); // {value: 1, done: false}
console.log(i.next()); // {value: 2, done: true}
console.log(i.next()); // {value: undefined. done: true}
```

#### 委托生成器
将迭代器合二为一
```
function* createNumberIterator() {
    yield 1;
    yield 2;
    // 将这句return 改为yield 3后
    // 输出结果为 1, 2, 3, undefined, undefined, undefined
    return 3; 
}
function* createRepeatingIterator(count) {
    for (let i = 0; i < count; i++) {
        yield "repeat";
    }
}
function* createCombinedIterator() {
    let result = yield* createNumberIterator();
    yield* createRepeatingIterator(result);
}
let iterator = createCombinedIterator();
console.log(iterator.next()); //{ value: 1, done: false }
console.log(iterator.next()); //{ value: 2, done: false }
console.log(iterator.next()); //{ value: 'repeat', done: false }
console.log(iterator.next()); //{ value: 'repeat', done: false }
console.log(iterator.next()); //{ value: 'repeat', done: false }
console.log(iterator.next()); //{ value: undefined, done: true }
```
如果想输出数值3，则
```
function* createCombinedIterator() {
    let result = yield* createNumberIterator();
    yield result;
    yield* createRepeatingIterator(result);
}
```

#### 异步任务执行
*  简单版
```
function run(taskDef) {
  //创建一个无使用限制的迭代器
    let task = taskDef();
    //开始执行任务
    let result = task.next();
    //循环调用next()的函数
    function step() {
        //如果任务未完成，则继续执行
        if (!result.done) {
            result = task.next(result.value);
            step();
        }
    }
    step();
}
run(function* (){
    let value = yield 1;
    console.log(value); // 1
    value = yield value +3;
    console.log(value); // 4
});
```
* 异步过程
```
let fs = require('fs');
function run(taskDef) {
    //创建一个无使用限制的迭代器
    let task = taskDef();
    //开始执行任务
    let result = task.next();
    //循环调用next()的函数
    function step() {
        //如果任务未完成，则继续执行
        if (!result.done) {
          //这里假设如果result.value是一个函数，那么就一定是异步执行的。
          if (typeof result.value === 'function') {
              //传入一个回调函数作为参数来调用它，回调函数遵循Node.js中有关
              //错误的约定：所有可能的错误放在第一个参数err中，结果放在第二个参数中。
              result.value(function (err, data) {
                  //err存在，说明执行过程产生了错误
                  if (err) {
                      result = task.throw(err);
                      return;
                  }
                  //如果没有错误产生，data就会被复制给yield左边的变量
                  result = task.next(data);
                  step();
              })
          } else {
              result = task.next(result.value);
              step();
            }
        }
    }
    step();
}
//readFile必须返回一个函数，接收callback
function readFile(filename) {
    return function (callback) {
        fs.readFile(filename, callback);
    }
}
run(function* () {
    let contents = yield readFile("txt.text");
    console.log(contents); //类似这样的结果 <Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>
});
```