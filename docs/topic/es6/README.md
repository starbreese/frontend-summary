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

## es6字符串
### es6中的标签模板
标签模板是让字符串模板跟在函数名后面，该函数来处理字符串模板：
```
let age = 22;
var tag = function(arr,arg){
    console.log(arr); // ['my age is', '']
    console.log(arg); // 22
}
tag`my age is${ age }`;
```
返回函数
```
function template(strings, ...keys) {
  return (function(...values) {
    var dict = values[values.length - 1] || {};
    var result = [strings[0]];
    keys.forEach(function(key, i) {
      var value = Number.isInteger(key) ? values[key] : dict[key];
      result.push(value, strings[i + 1]);
    });
    return result.join('');
  });
}
var t1Closure = template`${0}${1}${0}!`;
t1Closure('Y', 'A');  // "YAY!" 
var t2Closure = template`${0} ${'foo'}!`;
t2Closure('Hello', {foo: 'World'});  // "Hello World!"
```
### 原始字符串
```
var str = String.raw`Hi\n${2+3}!`;
// "Hi\n5!"
str.length;
// 6
str.split('').join(',');
// "H,i,\,n,5,!"
```

## es6函数
### 默认参数表达式
初次解析函数声明不会调用getValue函数，只有调用add函数且第二个参数不传入才会调用
```
let value = 5;
function getValue() {
    return ++value;
}
function add(first, second=getValue()){
    return first + second;
}
console.log(add(1, 1)) // 2
console.log(add(1)) // 7
console.log(add(1)) // 8
```

在引用参数默认值的时候，只允许引用前面参数的值，即先定义的参数不能访问后定义的参数
```
function add (first = second, second) {
    return first + second;
}
console.log(add(1, 1)) // 2
console.log(add(undefined, 1)) // 抛出错误
```

### 判断是否通过new关键字被调用
```
function Person(name) {
    if(this instanceof Person) {
        this.name = name
    } else {
        throw new Error("必须通过new关键字来调用Person")
    }
}
var person = new Person('a);
var notAPerson = Person.call(person, "bb"); // 有效
```

增加了new.target
```
function Person(name) {
    if(typeof new.target !== 'undefined') {
        this.name = name
    } else {
        throw new Error("必须通过new关键字来调用Person")
    }
}
```
### 箭头函数
#### 区别
* 没有this、super、arguments、new.target的绑定
* 不能通过new关键字调用
* 没有原型
* 不可以改变this的绑定, 取决于该函数外部非剪头函数的this值
* 没有auguments绑定，始终可以访问外围函数的arguments对象
* 不支持重复的命名函数

#### 注意
1. 返回一个对象
```
let getItem = id => ({id, name: 'temp'})
```

## 对象扩展
### 可计算属性
```
var suffix = " name";
var person = {
    ['first' + suffix]: "Nicholad",
    ['last' + suffix]: "Zakas"
}
console.log(person["first name"])
```

### 新增方法
#### Object.is()
```
Object.is(+0, -0) // false
+0===-0 //true
Object.is(NaN, NaN) // true
NaN===NaN // false
```
#### Object.assign()
* 可以接受任意数量的源对象，后面的会覆盖前面的
* 不能将提供者的访问器属性复制到接收对象中

### 自有属性枚举顺序
* Object.getOwnPropertyNames & Refelct.ownKeys顺序按枚举顺序
* for-in & Object.keys() & JSON.stringify()顺序不明确
* 枚举顺序规则
    1. 所有数字键按升序
    2. 所有字符串键按照被放入对象顺序排序
    3. 所有Symbol键按照被放入对象顺序排序

### 增强对象原型
* object.setPrototyoeOf()
* super访问对象原型
```
let friend = {
    getGreeting() {
        // 与Object.getPrototypeOf(this).getGreeting.call(this)相同
        return super.getGreeting() + ", hi!"
    }，
    //这样写会导致语法错误
    getGreeting: function() {
        return super.getGreeting() + ", hi!"
    }
}
```

super引用不是动态变化的，举例

```
let person = {
    getGreeting() {
        return "hello!"
    }
}
let friend = {
    getGreeting() {
        // Object.getPrototypeOf(this).getGreeting.call(this) 这种写法
        // 导致递归调用触发栈溢出报错
        return super.getGreeting() + ", hi!"
    }
}
Object.setPrototypeOf(friend, person);
let relative = Object.create(friend);
console.log(relative.getGreeting())
```

## 类
### 基本写法
* es5
```
function Point(x, y) {
    this.x = x;
    this.y = y;
}
Point.prototype.sayPoint = function() {
    console.log(`x: ${this.x}, y: ${this.y}`);
}
Object.keys(Point.prototype);  //['sayPoint']
```
* es6
```
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
    sayPoint() {
        console.log(`x: ${this.x}, y: ${this.y}`);
    }
}
Object.keys(Point.prototype);  //[]
```

### 区别
* 函数声明可以被提升，类声明不可被提升
* 类声明中的所有代码自动运行在严格模式下
* 类中所有方法都是不可枚举的
* 只能new关键字调用类的构造函数，否则报错
* 在类中不可修改类名
* es5不支持内置对象的继承，extends可以做到

### es5实现class
```
// 直接等价于 PersonClass
let PersonType2 = (function() {
    "use strict";
    //确保在类的内部不可以重写类名
    const PersonType2 = function(name) {
    // 确认函数被调用时使用了 new
        if (typeof new.target === "undefined") {
            throw new Error("Constructor must be called with new.");
        }
        this.name = name;
    }
    Object.defineProperty(PersonType2.prototype, "sayName", {
     value: function() {
        // 确认函数被调用时没有使用 new
        if (typeof new.target !== "undefined") {
            throw new Error("Method cannot be called with new.");
        }
        console.log(this.name);
    },
    //定义为不可枚举
    enumerable: false,
    writable: true,
    configurable: true
    });
    return PersonType2;
}());
```

### 类表达式
```
//声明式
class B {
  constructor() {}
}
//匿名表达式
let PersonClass = class {
    // 等价于 PersonType 构造器
    constructor(name) {
    this.name = name;
    }
    // 等价于 PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }
};
let person = new PersonClass("Nicholas");
person.sayName(); // 输出 "Nicholas"
console.log(person instanceof PersonClass); // true
console.log(person instanceof Object); // true
console.log(typeof PersonClass); // "function"
console.log(typeof PersonClass.prototype.sayName); // "function"
//命名表达式，B可以在外部使用，而B1只能在内部使用
let PersonClass = class PersonClass2 {
    // 等价于 PersonType 构造器
        constructor(name) {
    this.name = name;
    }
    // 等价于 PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }
};
console.log(typeof PersonClass); // "function"
console.log(typeof PersonClass2); // "undefined",只有在类内部才可以访问到
``` 

### 类的一些用法
1. 作为参数传入函数
```
function createObject(classDef) {
    return new classDef();
}
let obj = createObject(class {
    sayHi() {
        console.log("Hi!");
    }
});
obj.sayHi(); // "Hi!"
```
2. 通过立即调用类构造函数可以创建单例
```
//使用  new  来配合类表达式，并在表达式后面添加括号
let person = new class {
    constructor(name) {
        this.name = name;
    }
    sayName() {
        console.log(this.name);
    }
}("Nicholas");
person.sayName(); // "Nicholas"
```
3. 自有属性需要在类构造器中创建，而类还允许你在原型上定义访问器属性
```
class CustomHTMLElement {
    constructor(element) {
        this.element = element;
    }
    get html() {
        return this.element.innerHTML;
    }
    set html(value) {
        this.element.innerHTML = value;
    }
}
var descriptor = Object.getOwnPropertyDescriptor(CustomHTMLElement.prototype, "html");
console.log("get" in descriptor); // true
console.log("set" in descriptor); // true
console.log(descriptor.enumerable); // false
```
4. 可计算成员名称
```
let methodName = "sayName";
class PersonClass {
    constructor(name) {
        this.name = name;
    }
    [methodName]() {
        console.log(this.name);
    }
    get [methodName]() {
        return this.element.innerHTML;
    }
    set [methodName](value) {
       this.element.innerHTML = value;
    }
}
let me = new PersonClass("Nicholas");
me.sayName(); // "Nicholas"
```
5. 生成器方法
```
class MyClass {
    *createIterator() {
        yield 1;
        yield 2;
        yield 3;
    }
}
let instance = new MyClass();
let iterator = instance.createIterator();
```
6. 静态成员
* es5实现，将方法添加到构造函数中来模拟静态成员
```
function PersonType(name) {
    this.name = name;
}
// 静态方法
PersonType.create = function(name) {
    return new PersonType(name);
};
// 实例方法
PersonType.prototype.sayName = function() {
    console.log(this.name);
};
var person = PersonType.create("Nicholas");
```
* es6
不可在实例中访问静态成员，必须直接在类中访问
```
class PersonClass {
    // 等价于 PersonType 构造器
    constructor(name) {
        this.name = name;
    }
    // 等价于 PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }
    // 等价于 PersonType.create
    static create(name) {
        return new PersonClass(name);
    }
}
let person = PersonClass.create("Nicholas");
```

### 继承与派生类
#### es5 继承
```
function Super(name, age) {
    this.name = name;
    this.age = age;
}
function Sub(name, age, sex) {
    Super.call(this, name, age);
    this.sex = sex;
}
// 原型继承
Sub.prototype = new Super();
// 构造函数指向
Sub.prototype.constructor = Sub;
```
#### es6 extends
```
class A {
}
class B extends A {
    constructor(x, y, color) {
        super(x,y);
        this.color = color
    }
}
B.__proto__ === A    // true
B.prototype.__proto__ === A.prototype  // true
```
* 指定构造函数必须调用super(),并且在访问this前调用，只能在派生类的构造函数中使用
* 如果不想调用super(), 则唯一的方法是让类的构造函数返回一个对象，也就没有继承关系了
* super可以当作对象使用，ColorPoint.prototype
* 父类的静态方法也会被子类继承
* 只要表达式可以被解析成一个函数且具有[[constructor]]属性和原型，就可以使用extends派生
* mixin函数代替传统的继承方法
```
let SerializableMixin = {
    serialize() {
        return JSON.stringify(this);
    }
};
let AreaMixin = {
    getArea() {
        return this.length * this.width;
    }
};
function mixin(...mixins) {
    var base = function() {};
    Object.assign(base.prototype, ...mixins);
    return base;
}
class Square extends mixin(AreaMixin, SerializableMixin) {
    constructor(length) {
        super();
        this.length = length;
        this.width = length;
    }
}
var x = new Square(3);
console.log(x.getArea()); // 9
console.log(x.serialize()); // "{"length":3,"width":3}"
```