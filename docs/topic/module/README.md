## commonJS
模块系统需要**同步**读取模块文件内容，并编译执行以得到模块接口。

CommonJS模块导入用require，导出用module.exports。导出的对象需注意，如果是静态值，而且非常量，后期可能会有所改动的，请使用函数动态获取，否则无法获取修改值。导入的参数，是可以随意改动的，所以大家使用时要小心
* CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引，一旦输出，模块内部变化就影响不到这个值。
* CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。
* CommonJS 模块的顶层this指向当前模块
* 所有代码都运行在模块作用域。
* 模块可以多次加载，但只在第一次加载时运行一次，其结果会被缓存。后面再加载的时候，就直接读取缓存结果。要想让模块再运行的话，必须清除缓存。（delete require.cache[moduleName]）
* 模块加载的顺序，按照其在代码中出现的顺序。
* module对象：
module.id：模块的标识符，通常是带有绝对路径的模块文件名。
module.filename： 模块的文件名，带有绝对路径。
module.loaded：模块是否已经加载完成。
module.parents：[],调用该模块的模块。node下执行，会得到null或者undefined，require引入的会返回它的调用模块。可以用此判断该模块是不是入口模块。
module.exports：表示模块对外输出的值。还有个exports变量，指向的是module.exports。不能直接将 exports 变量指向一个值，因为这样等于切断了 exports 与 module.exports 的联系。
* require:用于加载模块文件。
（1）步骤：读入并执行一个js文件，返回该模块的exports对象。若没有发现指定模块则报错。
（2）加载规则：加载文件的后缀默认是.js。
|require参数|加载规则|
|--|--|
|'/'开头|加载的是绝对路径的文件|
|'./'开头|加载的是相对路径的文件|
|不以/或者./开头|默认提供的核心模块（位于Node的系统安装目录中）或者一个位于各级node_modules目录的已安装模块|
|不以/或./开头，是一个路径|先找第一级的位置，再依次往后续找|
指定的模块没发现的话，node会尝试为文件名添加.js,.json,.node后缀来找。
require.resolve可以拿到require命令加载的确切文件名。
* 输出的是值的缓存。

**值拷贝例子**
```
// lib.js
var counter = 3;
function getCounter() {
    counter++;
    return counter
}
module.exports = {
    counter: counter,
    getCounter: getCounter,
};

// index.js
var mod = require('./lib');
console.log(mod.counter);  // 3
console.log(mod.getCounter()); // 4
console.log(mod.getCounter()); // 5
console.log(mod.counter); // 3
```
输出结果
```
3
4
5
3
```
上面代码说明，lib.js模块加载以后，它的内部变化就影响不到输出的mod.counter了。这是因为mod.counter是一个原始类型的值，会被缓存。除非写成一个函数，才能得到内部变动后的值。

可将counter属性改成一个取数函数：可动态获得数据
```
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  get counter() {
    return counter
  },
  incCounter: incCounter,
};
```
**循环依赖**
```
// a.js
console.log('a starting');
exports.done = false;
const b = require('./b.js');
console.log('in a, b.done = %j', b.done);
exports.done = true;
console.log('a done');

// b.js
console.log('b starting');
exports.done = false;
const a = require('./a.js');
console.log('in b, a.done = %j', a.done);
exports.done = true;
console.log('b done');

// main.js
console.log('main starting');
const a = require('./a.js');
const b = require('./b.js');
console.log('in main, a.done = %j, b.done = %j', a.done, b.done);
```
输出结果:
```
main starting
a starting
b starting
in b, a.done = false
b done
in a, b.done = true
a done
in main, a.done = true, b.done = true
```

## ES6模块
* 静态化。编译时就能确定模块的依赖关系；commonjs和AMD只能在运行时确定。引擎对脚本静态分析的时候，遇到模块加载命令import，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。
* es6模块不是对象，而是通过export命令显式指定输出的代码，再通过import命令输入。缺点：没法引用es6模块本身，因为它不是对象。
* 自动采用严格模式，所以严格模式的规则，es6模块要遵守。
* 可以出现在模块顶层的任何位置，不能处于块级作用域内，否则会报错。import命令有提升效果，会提升到整个的头部。
* import会执行所加载的模块，不可省略模块后缀名。
    ```
    import './test.js'; // 会直接加载并执行test.js的代码。
    ```
* 尽量不要把require和import在一个文件里混用。在一起用的话会先执行import的。
* es6 module可以继承
* ES6 模块不会缓存运行结果，而是动态地去被加载的模块取值，并且变量总是绑定其所在的模块。由于 ES6 输入的模块变量，只是一个“符号连接”，所以这个变量是只读的，对它进行重新赋值会报错。
* ES6 模块之中，顶层的this指向undefined

**值引用例子**
```
// lib.js
export let counter = 3;
export function incCounter() {
  counter++;
}
// main.js
import { counter, incCounter } from './lib';
console.log(counter); // 3
incCounter();
console.log(counter); // 4
```

## AMD规范
* 采用异步方式加载模块，模块的加载不影响它后面语句的运行
* 推崇依赖前置、提前执行
```
// 网页中引入require.js及main.js
<script src="js/require.js" data-main="js/main"></script>
// main.js,首先用config()指定各模块路径和引用名
require.config({
  baseUrl: "js/lib",
  paths: {
    "jquery": "jquery.min",  //实际路径为js/lib/jquery.min.js
    "underscore": "underscore.min",
  }
});
// 执行基本操作
require(["jquery","underscore"],function($,_){
  // some code here
});
```
引用模块的时候，我们将模块名放在[]中作为reqiure()的第一参数；如果我们定义的模块本身也依赖其他模块,那就需要将它们放在[]中作为define()的第一参数。
```
// 定义math.js模块
define(function () {
    var basicNum = 0;
    var add = function (x, y) {
        return x + y;
    };
    return {
        add: add,
        basicNum :basicNum
    };
});
// 定义一个依赖underscore.js的模块
define(['underscore'],function(_){
    var classify = function(list){
        _.countBy(list,function(num){
            return num > 30 ? 'old' : 'young';
        })
    };
    return {
        classify :classify
    };
})
// 引用模块，将模块放在[]内
require(['jquery', 'math'],function($, math){
    var sum = math.add(10,20);
    $("#sum").html(sum);
});
```

## CMD规范
* 依赖就近、延迟执行
```
/** AMD写法 **/
define(["a", "b", "c", "d", "e", "f"], function(a, b, c, d, e, f) { 
     // 等于在最前面声明并初始化了要用到的所有模块
    a.doSomething();
    if (false) {
        // 即便没用到某个模块 b，但 b 还是提前执行了
        b.doSomething()
    } 
});
/** CMD写法 **/
define(function(require, exports, module) {
    var a = require('./a'); //在需要时申明
    a.doSomething();
    if (false) {
        var b = require('./b');
        b.doSomething();
    }
});
/** sea.js **/
// 定义模块 math.js
define(function(require, exports, module) {
    var $ = require('jquery.js');
    var add = function(a,b){
        return a+b;
    }
    exports.add = add;
});
// 加载模块
seajs.use(['math.js'], function(math){
    var sum = math.add(1+2);
});
```
## UMD（Universal Module Definition)
* 希望提供一个前后端跨平台的解决方案。
1. 以此判断是否支持Nodejs模块格式（typeof exports === 'object'）
2. 是否支持AMD（typeof define === 'function'）,
3. 都不支持则将模块公开到全局（window或global）
```
// if the module has no dependencies, the above pattern can be simplified to
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD. Register as an anonymous module.
        define([], factory);
    } else if (typeof exports === 'object') {
        // Node. Does not work with strict CommonJS, but
        // only CommonJS-like environments that support module.exports,
        // like Node.
        module.exports = factory();
    } else {
        // Browser globals (root is window)
        root.returnExports = factory();
  }
}(this, function () {
    // Just return a value to define the module export.
    // This example returns an object, but the module
    // can return a function as the exported value.
    return {};
}));
```
## 对比
|模块规范|导出命令|导入命令|静态还是动态|值是否被缓存|是否采用严格模式|加载是否是同步的|适应端|顶层this指向哪里
|--|--|--|--|--|--|--|--|--|
|es6 module|export|import|静态|值的引用，不会缓存，运行时再根据引用去取值|是||浏览器和服务端|undefined|
|commonjs|module.exports|require|动态，运行时加载|会缓存||同步|服务端|指向当前模块|
|AMD规范|return或者兼容commonjs的|||||异步|浏览器|