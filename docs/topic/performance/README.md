## 参考资料
## 性能指标
## 优化方案
    1. 评估实际经验和设置合适的目标。一个很好的目标是追求首次有意义的渲染时间 < 1 秒，同时 Speed Index < 1250 秒，慢速 3G 网络下首次可交互时间 < 5秒，TTI < 2 秒。针对渲染时间和首次可交互时间做优化。
    2. 为你的主要模板准备关键 CSS，并在放在页面的 head 标签内（预算应小于 14 KB）。对于 CSS/JS，使它们小于关键文件大小最大预算 gzipped 压缩后为 170 KB（未压缩为 0.7 MB）。
    3. 尽可能地让更多的脚本分割，优化，defer 加载或者懒加载，检查轻量级的可选包并限制第三方包的大小。
    4. 使用 <script type="module"> 来让代码只对旧浏览器工作。
    5. 试着整个 CSS 规则并测试 in-body CSS。
    6. 使用更快的 dns-lookup，preconnect，prefetch 和 preload 来添加资源提示来加速分发。
    7. 给网络字体分组并异步加载，在 CSS 中利用 font-display 来加速首次渲染。
    8. 优化图片，并考虑为重要的页面（例如首页）使用 WebP。
    9. 检查 HTTP 头设置的缓存并确保已经被合适地设置。
    10. 在服务器上启用 Brotli 和 Zopfli 压缩。（如果不能，别忘了启用 Gzip 压缩。）
    11. 如果 HTTP/2 可用，启用 HPACK 压缩并开始监控 mixed-content 警告。开启 OSCP 压缩。
    12. 在 service worker 中缓存字体，样式，JavaScript 和图片等资源文件。
### vue优化
1. 使用 Object.freeze()，这样被设置为不可配置之后的对象属性时，不会为对象加上 setter getter 等数据劫持的方法。比较适合展示类的场景，如果你的数据属性需要改变，可以重新替换成一个新的 Object.freeze()的对象。
2. vue启用生产环境模式 [官方说明](https://cn.vuejs.org/v2/guide/deployment.html)
3. 合理使用持久化 Store 数据，尽量减少直接写入 Storage 的频率
    * 多次写入操作合并为一次，比如采用函数节流或者将数据先缓存在内存中，最后在一并写入
    * 只有在必要的时候才写入，比如只有关心的模块的数据发生变化的时候才写入
4. 优化无限列表性能
    * vue-virtual-scroll-list 和 vue-virtual-scroller
    * [IntersectionObserver API](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver)DOM 回收、墓碑元素和滚动锚定
    * [react版本](https://codesandbox.io/s/sad-darkness-uk1wg?file=/src/SlidingWindowScroll.js)
5. 通过组件懒加载优化超长应用内容初始渲染性能
    * v-if
    * 懒加载组件
6. ssr或者预渲染
    * 设置预渲染更简单，并可以将你的前端作为一个完全静态的站点
7. 控制 js 代码的体积以及按需加载
    * tree-shaking(消除无用的js代码) : rollup + uglify和 webpack + uglify
### JavaScript执行
1. 使用 requestAnimationFrame 来更新页面
2. 使用 Web Worker 来处理复杂的计算
3. 使用 transform 和 opacity 来完成动画，不需要经历 layout 和 paint 过程