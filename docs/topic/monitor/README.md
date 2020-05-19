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
* 网络性能
* 内存
