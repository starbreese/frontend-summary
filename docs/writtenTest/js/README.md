1. this作用域
    ```
var name="abc";
var person={
    name:'cba',
    getName:function () {
        return this.name;
    }
}
console.log(person.getName()); // cba
var p1=person.getName;
console.log(p1()); // abc
var p2=new p1();
console.log(p2.name); // undefined
```
    **result: cba, abc, undefined**
2. call 和闭包
    ```
var v='cba';
(function () {
    var v='abc'
    function A() {
        var v='a'
        this.getV=function () {
            console.log(v);
        }
    }
    function B() {
        var v='b' 
        A.call(this);
    }
    var b=new B();
    b.getV(); // a
}())
```
    **result: a**

    换成
    ```
function A() {
    var v='a'
    this.getV=function () {
        console.log(this.v);
    }
}
function B() {
    this.v='b' 
    A.call(this);
}
```
    **result: b**

    换成
    ```
function A() {
    this.v='a'
    this.getV=function () {
        console.log(this.v);
    }
}
function B() {
    this.v='b'
    A.call(this);
}
```
    **result: a**
3. 定时器
```
var i=1;
(
    function () {
        var s = new Date().getTime();
        var si = setInterval(function () {
            var n =  new Date().getTime();
            if(n <(s+200)){
                i++;
            }else{
                console.log(i);
                clearInterval(si);
            }
        },10)
    }
)()
```
    **result:**
    20=200/10

4. 作用域
```
var foo=1;
function main() {
    console.log(foo);
    var foo=2;
    console.log(this.foo);
    this.foo=3;
}
main();
new main();
```
    **result:**
    
    undefined 1<br>
    main:{foo: 3}
5. 作用域
    ```
function one() {
    this.name = 1;
    function two() {
        this.name = 2;
        function three() {
            var name = 3;
            console.log(this.name);
        }
        return three;
    }
    return two;
}
one()()();
```
    **result: 2**
    
    换成<br>
    ```
function one() {
    this.name = 1;
    function two() {
        this.name = 2;
        function three() {
            this.name = 3;
            console.log(this.name);
        }
        return three;
    }
    return two;
}
one()()();
```
  **result: 3**
6. 异步 <https://juejin.im/post/58f1fa6a44d904006cf25d22>
    ```
for (var i = 0; i < 5; i++) {
    setTimeout(function() {
        console.log(new Date, i);
    }, 2000);
}
console.log(new Date, i);
```
    **result**
    ```nohighlight
    2017-03-18T00:43:44.873Z 5
    2017-03-18T00:43:46.866Z 5
    2017-03-18T00:43:46.868Z 5
    2017-03-18T00:43:46.868Z 5
    2017-03-18T00:43:46.868Z 5
    2017-03-18T00:43:46.868Z 5
    ```
    循环执行过程中，几乎同时设置了 5 个定时器，一般情况下，这些定时器都会在 1 秒之后触发，而循环完的输出是立即执行的
#### 期望输出5->0,1,2,3,4
    ##### 第一种
    ```
    for(var i = 0; i < 5; i++) {
        (function(j){
            setTimeout(function(){
                console.log(new Date, j)
            }, 1000)
        })(i)
    }
    console.log(new Date, i);
    ```
    ##### 第二种
    ```
    for (var i = 0; i < 5; i++) {
        setTimeout(function(j) {
            console.log(new Date, j);
        }, 1000, i);
    }
    console.log(new Date, i);
    ```
    ##### 第三种
    ```
    var output = function (i) {
        setTimeout(function() {
            console.log(new Date, i);
        }, 1000);
    };
    for (var i = 0; i < 5; i++) {
        output(i);  // 这里传过去的 i 值被复制了
    }
    console.log(new Date, i);
    ```
    ##### 第四种
    ```
    for (let i = 0; i < 5; i++) {
        setTimeout(function() {
            console.log(new Date, i);
        }, 1000);
    }
    console.log(new Date, i);
    ```
#### 期望输出0->1->2->3->4->5, 每隔一秒输出
##### 第一版
```
for (var i = 0; i < 5; i++) {
    (function(j) {
        setTimeout(function() {
            console.log(new Date, j);
        }, 1000 * j);  // 这里修改 0~4 的定时器时间
    })(i);
}
setTimeout(function() { // 这里增加定时器，超时设置为 5 秒
    console.log(new Date, i);
}, 1000 * i);
```
##### 第二版
```
const tasks = [];
for (var i = 0; i < 5; i++) {   // 这里 i 的声明不能改成 let，如果要改该怎么做？
    ((j) => {
        tasks.push(new Promise((resolve) => {
            setTimeout(() => {
                console.log(new Date, j);
                resolve();  // 这里一定要 resolve，否则代码不会按预期 work
            }, 1000 * j);   // 定时器的超时时间逐步增加
        }));
    })(i);
}
Promise.all(tasks).then(() => {
    setTimeout(() => {
        console.log(new Date, i);
    }, 1000);   // 注意这里只需要把超时设置为 1 秒
});
```
#####  第三版
```
const tasks = []; // 这里存放异步操作的 Promise
const output = (i) => new Promise((resolve) => {
    setTimeout(() => {
        console.log(new Date, i);
        resolve();
    }, 1000 * i);
});
// 生成全部的异步操作
for (var i = 0; i < 5; i++) {
    tasks.push(output(i));
}
// 异步操作完成之后，输出最后的 i
Promise.all(tasks).then(() => {
    setTimeout(() => {
        console.log(new Date, i);
    }, 1000);
});
```
##### 第四版
```
// 模拟其他语言中的 sleep，实际上可以是任何异步操作
const sleep = (timeountMS) => new Promise((resolve) => {
    setTimeout(resolve, timeountMS);
});
(async () => {  // 声明即执行的 async 函数表达式
    for (var i = 0; i < 5; i++) {
        if (i > 0) {
            await sleep(1000);
        }
        console.log(new Date, i);
    }
    await sleep(1000);
    console.log(new Date, i);
})();
```

7. 实现一个getX, 支持path取数，支持函数，数组
```javascript
    function getx(source, path) {
        // \d+可表示多位数字
        const paths = path.replace(/\[(\d+)\]/g, '.$1').split('.');
        // 匹配() .+?
        const reg= /\((.*?)\)/g;

        let result = source;
        for (const p of paths) {
            if(reg.test(p)){ //如果匹配上直接获取括号里的内容
                let args = RegExp.$1;
                args = args ? args.split(',') : [];
                let regNum = /\d+/g;
                args = args.map(a => {
                    if(regNum.test(a)) {
                        return parseInt(a);
                    } else {
                        return a
                    }
                });
                let funcName = p.slice(0, p.indexOf('('));
                result = result[funcName](...args);
            } else {
                result = result[p];
            }
            if (result === undefined) {
                return undefined
            }
            }
        return result
    }
    console.log(getx({a:{b:{c:100}}}, 'a.b.c'));
    console.log(getx({a:{b:{c:100}}}, 'a.b.c.d'));
    console.log(getx({a:{b:[{c:100}]}}, 'a.b[0].c'));
    console.log(getx({a:{b:{c:function(){return {d:100}}}}} , 'a.b.c().d'));
    console.log(getx({a:{b:{c:function(factor){return {d:100 * factor}}}}} , 'a.b.c(10).d'));
    console.log(getx({a:{b:{c:function(x, y){return {d: x * y}}}}} , 'a.b.c(10, 20).d'));
```
 