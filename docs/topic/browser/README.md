## 浏览器渲染
### 渲染过程
1. 解析HTML，生成DOM树，解析CSS，生成CSSOM树
2. 将DOM树和CSSOM树结合，生成渲染树(Render Tree)
3. Layout(回流):根据生成的渲染树，进行回流(Layout)，得到节点的几何信息（位置，大小）
4. Painting(重绘):根据渲染树以及回流得到的几何信息，得到节点的绝对像素
5. Display:将像素发送给GPU，展示在页面上。
* 渲染树中不可见节点：
  1. 一些不会渲染输出的节点，比如script、meta、link等。
  2. 一些通过css进行隐藏的节点。比如display:none。注意，利用visibility和opacity隐藏的节点，还是会显示在渲染树上的。只有display:none的节点才不会显示在渲染树上。

### 何时触发回流和重绘
**回流一定会触发重绘，而重绘不一定会回流**
#### 触发回流
1. 添加或删除可见的DOM元素
2. 元素的位置发生变化
3. 元素的尺寸发生变化（包括外边距、内边框、边框大小、高度和宽度等）
4. 内容发生变化，比如文本变化或图片被另一个不同尺寸的图片所替代。
5. 页面一开始渲染的时候（这肯定避免不了）
6. 浏览器的窗口尺寸变化（因为回流是根据视口的大小来计算元素的位置和大小的）

### 渲染优化
1. 当你获取布局信息的操作的时候，会强制队列刷新，慎用以下属性：
  * offsetTop、offsetLeft、offsetWidth、offsetHeight
  * scrollTop、scrollLeft、scrollWidth、scrollHeight
  * clientTop、clientLeft、clientWidth、clientHeight
  * getComputedStyle()
  * getBoundingClientRect
2. 合并多次对DOM和样式的修改，然后一次处理掉
  * 使用cssText
  * 修改CSS的class
3. 批量修改DOM（使元素脱离文档流；对其进行多次修改；将元素带回到文档中）**display**
4. 避免触发同步布局事件
```
// before
function initP() {
    for (let i = 0; i < paragraphs.length; i++) {
        paragraphs[i].style.width = box.offsetWidth + 'px';
    }
}
// after
const width = box.offsetWidth;
function initP() {
    for (let i = 0; i < paragraphs.length; i++) {
        paragraphs[i].style.width = width + 'px';
    }
}
```
在每次循环的时候，都读取了box的一个offsetWidth属性值，然后利用它来更新p标签的width属性。这就导致了每一次循环的时候，浏览器都必须先使上一次循环中的样式更新操作生效，才能响应本次循环的样式读取操作。每一次循环都会强制浏览器刷新队列。
5. 对于复杂动画效果,使用绝对定位让其脱离文档流
6. 使用css3硬件加速（GPU加速），可以让transform、opacity、filters、Will-change这些动画不会引起回流重绘

## 浏览器缓存
* 浏览器每次发起请求，都会先在浏览器缓存中查找该请求的结果以及缓存标识
* 浏览器每次拿到返回的请求结果都会将该结果和缓存标识存入浏览器缓存中

### 强制缓存
* 强制缓存就是向**浏览器缓存**查找该请求结果，并根据该结果的缓存规则来决定是否使用该缓存结果的过程，强制缓存的情况主要有三种
* 控制强制缓存的字段分别是Expires和Cache-Control，其中Cache-Control优先级比Expires高。
* 浏览器读取缓存的顺序为memory –> disk
#### 内存缓存(from memory cache)和硬盘缓存(from disk cache)

* 内存缓存
  1. 快速读取：内存缓存会将编译解析后的文件，直接存入该进程的内存中，占据该进程一定的内存资源，以方便下次运行使用时的快速读取。
  2. 时效性：一旦该进程关闭，则该进程的内存则会清空
* 磁盘缓存
  1. 硬盘缓存则是直接将缓存写入硬盘文件中，读取缓存需要对该缓存存放的硬盘文件进行I/O操作，然后重新解析该缓存内容，读取复杂，速度比内存缓存慢
* 在浏览器中，浏览器会在js和图片等文件解析执行后直接存入内存缓存中，那么当刷新页面时只需直接从内存缓存中读取(from memory cache)；而css文件则会存入硬盘文件中，所以每次渲染页面都需要从硬盘读取缓存(from disk cache)。
#### Expires
Expires是HTTP/1.0控制网页缓存的字段，其值为服务器返回该请求结果缓存的到期时间，即再次发起该请求时，如果客户端的时间小于Expires的值时，直接使用缓存结果
#### cache-control
在HTTP/1.1中，Cache-Control是最重要的规则，主要用于控制网页缓存，主要取值为：
  * public：所有内容都将被缓存（客户端和代理服务器都可缓存）
  * private：所有内容只有客户端可以缓存，Cache-Control的默认取值
  * no-cache：客户端缓存内容，但是是否使用缓存则需要经过协商缓存来验证决定
  * no-store：所有内容都不会被缓存，即不使用强制缓存，也不使用协商缓存
  * max-age=xxx (xxx is numeric)：缓存内容将在xxx秒后失效

### 协商缓存
  * 强制缓存失效后，浏览器携带缓存标识向**服务器**发起请求，由服务器根据缓存标识决定是否使用缓存的过程
  * 生效为304
  * 控制协商缓存的字段分别有：Last-Modified / If-Modified-Since和Etag / If-None-Match，其中Etag / If-None-Match的优先级比Last-Modified / If-Modified-Since高。
  
  #### Last-Modified/If-Modified-Since
  * Last-Modified**服务器**响应请求时，返回该资源文件在服务器最后被修改的时间
  * If-Modified-Since是**客户端**再次发起该请求时，携带上次请求返回的Last-Modified值，通过此字段值告诉服务器该资源上次请求返回的最后被修改时间。服务器收到该请求，发现请求头含有If-Modified-Since字段，则会根据If-Modified-Since的字段值与该资源在服务器的最后被修改时间做对比，若服务器的资源最后被修改时间大于If-Modified-Since的字段值，则重新返回资源，状态码为200；否则则返回304，代表资源无更新，可继续使用缓存文件
  #### Etag / If-None-Match
  * Etag:服务器响应请求时，返回当前资源文件的一个唯一标识(由服务器生成)
  * If-None-Match:客户端再次发起该请求时，携带上次请求返回的唯一标识Etag值，通过此字段值告诉服务器该资源上次请求返回的唯一标识值。服务器收到该请求后，发现该请求头中含有If-None-Match，则会根据If-None-Match的字段值与该资源在服务器的Etag值做对比，一致则返回304，代表资源无更新，继续使用缓存文件；不一致则重新返回资源文件，状态码为200

### 总结
强制缓存优先于协商缓存进行，若强制缓存(Expires和Cache-Control)生效则直接使用缓存，若不生效则进行协商缓存(Last-Modified / If-Modified-Since和Etag / If-None-Match)，协商缓存由服务器决定是否使用缓存，若协商缓存失效，那么代表该请求的缓存失效，重新获取请求结果，再存入浏览器缓存中；生效则返回304，继续使用缓存