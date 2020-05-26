## tapable的应用
[github地址](https://github.com/webpack/tapable/tree/tapable-1)

Webpack 本质上是一种事件流的机制，它的工作流程就是将各个插件串联起来，而实现这一切的核心就是 tapable，Webpack 中最核心的，负责编译的 Compiler 和负责创建 bundles 的 Compilation 都是 tapable 构造函数的实例
```
// 引入 tapable 如下
const {
    SyncHook,
    SyncBailHook,
    SyncWaterfallHook,
    SyncLoopHook,
    AsyncParallelHook,
    AsyncParallelBailHook,
    AsyncSeriesHook,
    AsyncSeriesBailHook,
    AsyncSeriesWaterfallHook
 } = require("tapable");
```
* 钩子类型分为三种:
    * 普通型basic：这个比较好理解就是按照tap的注册顺序一个个向下执行
    * 流水型water：虽然也是按照tap的顺序一个个向下执行，但是如果上一个tap有返回值，那么下一个tap的传入参数就是上一个tap的返回值。
    * 熔断型bail：如果返回了null以外的值，就不继续执行了。

* 钩子类型也可以分成同步和异步
    * Async 类型可以使用 tap、tapSync 和 tapPromise 注册不同类型的插件 “钩子”，分别通过 call、callAsync 和 promise 方法调用

## 打包优化
1. 配置 externals，工具库直接使用 cdn，不需要打包，例如：vue，vue-router 等
2. 在处理 loader 时，配置 include，缩小 loader 检查范围。
3. happypack，因nodejs是单线程执行编译，而happypack是启动node的多线程进行构建，进而提高构建速度
4. 用DllPlugin插件单独编译一些不经常改变的代码，比如node_modules的第三方库
5. 删除不需要的一些代码，利用SplitChunksPlugin 进行分块
6. cache-loader来进行缓存持久化
7. eslint代码校验其实是一个很费时间的一个步奏。可以把eslint的范围缩小到src,且只检查*.js 和 *.vue,生产环境不开启lint，使用pre-commit或者husky在提交前校验