## 监控指标
* crash
    1. performance.onresourcetimingbufferfull
    ```
    function buffer_full(event) {
        console.log("WARNING: Resource Timing Buffer is FULL!");
        performance.setResourceTimingBufferSize(200);
    }
    function init() {
        // Set a callback if the resource buffer becomes filled
        performance.onresourcetimingbufferfull = buffer_full;
    }
    <body onload="init()">
    ```
* 首屏时间
    * DOMContentLoaded
        
        当初始的 HTML 文档被完全加载和解析完成之后，DOMContentLoaded 事件被触发，而无需等待样式表、图像和子框架的完全加载。另一个不同的事件 load 应该仅用于检测一个完全加载的页面。 这里有一个常见的错误，就是在本应使用 DOMContentLoaded 会更加合适的情况下，却选择使用 load，所以要谨慎。注意：DOMContentLoaded 事件必须等待其所属script之前的样式表加载解析完成才会触发
    * first contentful paint (FCP)
        
        浏览器首次呈现有意义的任何内容的时间，该时间包括文本，前景或背景图像，画布或SVG。
    * first meaningful paint (FMP)
        
        反映主要内容出现在页面上所需的时间，也侧面反映了服务器输出任意数据的速度。FMP 时间过长一般意味着 JavaScript 阻塞了主线程，也有可能是后端/服务器的问题。
    * largest contentful paint (LCP)
        
        最大内容绘制（LCP）指标表示视口中可见的最大内容元素的渲染时间，light
    * time to interactive (TTI)
        
        在此时间点，页面布局已经稳定，主要的网络字体已经可见，主线程已可以响应用户输入 — 基本上意味着只是用户可以与 UI 进行交互。是描述“网站可正常使用前，用户所需要等待的时长”的关键因素。
    * 首次输入延迟（First Input Delay，FID 或 Input responsiveness）
        
        从用户首次与页面交互，到网站能够响应该交互的时间。与 TTI 相辅相成，补全了画像中缺少的一块：在用户切实与网站交互后发生了什么
    * 速度指数（Speed Index）[https://dev.to/borisschapira/web-performance-fundamentals-what-is-the-speed-index-2m5i]
        
        衡量视觉上页面被内容充满的速度，数值越低越好。速度指数由视觉上的加载速度计算而得，只是一个计算值。同时对视口尺寸也很敏感，因此你需要根据目标用户设定测试配置的范围。
    

* 网络性能
* 内存
* fps (frames per second)
    <60时，用户体验会好