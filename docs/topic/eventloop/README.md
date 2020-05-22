## 任务队列
浏览器的 Event Loop 遵循的是 HTML5 标准，而 NodeJs 的 Event Loop 遵循的是 libuv。
* 宏任务(可有多个)：script（全局任务）, setTimeout, setInterval, setImmediate, I/O, UI rendering
* 微任务(只有1个队列)：process.nextTick, Promise, Object.observer, MutationObserver

### 浏览器的eventloop
浏览器这边，共分3步：
1. 取一个宏任务来执行。执行完毕后，下一步。
2. 取一个微任务来执行，执行完毕后，再取一个微任务来执行。直到微任务队列为空，执行下一步。
3. 更新UI渲染。

Event Loop 会无限循环执行上面3步，这就是Event Loop的主要控制逻辑。其中，第3步（更新UI渲染）会根据浏览器的逻辑，决定要不要马上执行更新。毕竟更新UI成本大，所以，一般都会比较长的时间间隔，执行一次更新。

我们代码开始执行都是从script（全局任务）开始，所以，一旦我们的全局任务（属于宏任务）执行完，就马上执行完整个微任务队列。
```
console.log('script start');
// 微任务
Promise.resolve().then(() => {
    console.log('p 1');
});
// 宏任务
setTimeout(() => {
    console.log('setTimeout');
}, 0);
var s = new Date();
while(new Date() - s < 1000); // 阻塞1000ms
// 微任务
Promise.resolve().then(() => {
    console.log('p 2');
});
console.log('script ent');
```
**输出结果**
```
script start
script ent
p 1
p 2
setTimeout
```