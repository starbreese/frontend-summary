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

