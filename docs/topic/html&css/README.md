## html
### 触摸事件 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Touch_events)
触摸事件与鼠标事件类似，不同的是触摸事件还提供同一表面不同位置的同步触摸。TouchEvent 接口将当前所有活动的触摸点封装起来。Touch 接口表示单独一个触摸点，其中包含参考浏览器视角的相对坐标。

从事件的 TouchEvent.changedTouches 属性中获得已改变的触摸点列表

通过读取每个触摸点的 Touch.identifier 属性实现的。对每个触摸点而言，该属性是个唯一的整数，且手指接触表面的整个过程中，这个属性保持不变

由于在 touchstart（或系列 touchmove 事件里的首个）中调用 preventDefault() 也会阻止相应的鼠标事件的触发，因此一般情况下我们在touchmove 而不是 touchstart 中调用它，这样，鼠标事件仍可正常触发，链接等部件也可继续工作

## css基础

### box-sizing
* content-box  是默认值。如果你设置一个元素的宽为100px，那么这个元素的内容区会有100px 宽，并且任何边框和内边距的宽度都会被增加到最后绘制出来的元素宽度中。W3C的标准盒模型
* border-box 告诉浏览器：你想要设置的边框和内边距的值是包含在width内的。也就是说，如果你将一个元素的width设为100px，那么这100px会包含它的border和padding，内容区的实际宽度是width减去(border + padding)的值。大多数情况下，这使得我们更容易地设定一个元素的宽高。IE盒模型

## h5屏幕适配
### 基础概念
* 物理像素（physical pixel），也叫设备像素
* 分辨率（resolution）
    <br>iPhone手机的主屏：4.7英寸1334x750，就是指：对角线4.7英寸长，高1334个物理像素数，宽750个物理像素数。
* css像素
    <br>px 是一个 相对单位 ，相对的是 物理像素（physical pixel），这也就是说到底我们平常开发中的 1px 在每个屏幕上怎么显示，完全取决于这个设备！
* 设备像素比（device pixel ratio）
    * 在javascript中，可以通过 window.devicePixelRatio 获取到当前设备的 dpr。
    * 在css中，可以通过 -webkit-device-pixel-ratio，-webkit-min-device-pixel-ratio 和 -webkit-max-device-pixel-ratio 进行媒体查询，对不同dpr的设备，做一些样式适配
    * 当 dpr=2 时，表示：1个CSS像素 = 4个物理像素。因为像素点都是正方形，所以当1个CSS像素需要的物理像素增多2倍时，其实就是长和宽都增加了2倍。
* 像素密度（pixers per inch）
    * ppi pixers per inch，出现于计算机显示领域（当然也是Android中的习惯叫法）
    * dpi dots per inch，出现于打印或印刷领域（当然也是iOS中的习惯叫法）
    * 你计算 ppi 只能用对角线的物理像素数来除以对角线的实际单位
* 设备独立像素（density-independent pixel) dips or dp
* rem & em
    * rem是和根元素关联的，不依赖当前元素。不管你在文档中的什么地方使用这个单位，1.2rem的计算值是相等的，等于1.2倍的根元素的字号大小
    * rem不仅可以适用于字体，同样可以用于 width height margin 这些样式的单位
    * em是根据其父元素的字体大小来设置
    * 当你不确定的时候，对font-size使用rem，对border使用px，以及对其他大多数属性使用em。
### 元素自适应问题
在 1080px 的视觉稿中，左上角有个logo，宽度是 180px（高度问题同理可得）。
那么logo在不同的手机屏幕上等比例显示应该多大尺寸呢？
其实按照比例换算，我们大致可以得到如下的结果：

在CSS像素是 375px 的手机上，应该显示多大呢？结果是：375px * 180 / 1080 = 62.5px
在CSS像素是 360px 的手机上，应该显示多大呢？结果是：360px * 180 / 1080 = 60px
在CSS像素是 320px 的手机上，应该显示多大呢？结果是：320px * 180 / 1080 = 53.3333px

#### 用css的媒体查询 @media
```
@media only screen and (min-width: 375px) {
  .logo {
    width : 62.5px;
  }
}
@media only screen and (min-width: 360px) {
  .logo {
    width : 60px;
  }
}
@media only screen and (min-width: 320px) {
  .logo {
    width : 53.3333px;
  }
}
```
#### 使用 rem 单位
```
@media only screen and (min-width: 375px) {
  html {
    font-size : 375px;
  }
}
@media only screen and (min-width: 360px) {
  html {
    font-size : 360px;
  }
}
@media only screen and (min-width: 320px) {
  html {
    font-size : 320px;
  }
}
.logo{
	width : 180rem / 1080;
}
```
使用sass
```
@media only screen and (min-width: 375px) {
  html {
    font-size : 375px;
  }
}
@media only screen and (min-width: 360px) {
  html {
    font-size : 360px;
  }
}
@media only screen and (min-width: 320px) {
  html {
    font-size : 320px;
  }
}
//定义方法：calc
@function calc($val){
    @return $val / 1080;
}
.logo{
	width : calc(180rem);
}
```

**缺点**
* 针对不同的手机分辨率，需要写多套 @media 查询语句，多一种机型就需要多写一套查询语句，而且随着现在手机的层出不穷，这个页面很有可能在一些新出的机型上有问题。每个元素都需要除以1080这个设计稿的尺寸。
* 在设置html的font-size的时候一定要保证最小等于8px

#### js动态设置根字体
* 动态设置字体大小
```
//获取手机屏幕宽度
var deviceWidth = document.documentElement.clientWidth;
//将方案二中的media中的设置，在这里动态设置
//这里设置的就是html的font-size
document.documentElement.style.fontSize = deviceWidth + 'px';
```
* 在html中设置了如下标签：<meta name="viewport" content="width=device-width">,要不然获取到的结果将始终是：980
* sass代码
```
//定义方法：calc
@function calc($val){
    @return $val / 1080;
}
.logo{
	width : calc(180rem);
}
```
### 1px
* 使用css3的 scaleY(0.5) 来解决
```
.div:before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  bottom: auto;
  right: auto;
  height: 1px;
  width: 100%;
  background-color: #c8c7cc;
  display: block;
  z-index: 15;
  -webkit-transform-origin: 50% 0%;
          transform-origin: 50% 0%;
}
@media only screen and (-webkit-min-device-pixel-ratio: 2) {
  .div:before {
    -webkit-transform: scaleY(0.5);
            transform: scaleY(0.5);
  }
}
@media only screen and (-webkit-min-device-pixel-ratio: 3) {
  .div:before {
    -webkit-transform: scaleY(0.33);
            transform: scaleY(0.33);
  }
}
```
* 页面缩放解决问题



### 文字rem问题
针对不同的dpr设置具体的字体
```
.title {
    font-size: 12px;
}
[data-dpr="2"] .title {
    font-size: 24px;
}
[data-dpr="3"] .title {
    font-size: 36px;
}
```

### 横竖屏问题
* width=height
```
var deviceWidth = document.documentElement.clientWidth,
    deviceHeight = document.documentElement.clientHeight
//横屏状态
if (window.orientation === 90 || window.orientation === -90) {
    deviceWidth = deviceHeight;
};
//设置根字体大小
document.documentElement.style.fontSize = deviceWidth + 'px';
```

### 最终版本适配 [参考](https://github.com/amfe/article/issues/17)
* html
```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width">
</head>
<body>
    <!-- 正文 -->
</body>
</html>
```
* js
```
/**
 * @DESCRIPTION 移动端页面适配解决方案 v1.0
 * @AUTHOR      Night
 * @DATE        2018年08月01日
 */
(function(window, document){
    var docEle = document.documentElement,
        dpr    = window.devicePixelRatio || 1,
        scale  = 1 / dpr;
    var fontSizeRadio = 1, //手机字体正常比例
        isLandscape   = false;//是否横屏
    ///////////////////////// viewport start //////////////////////////////////
    //设置页面缩放比例并禁止用户手动缩放
    document.getElementsByName('viewport')[0].setAttribute('content','width=device-width,initial-scale='+scale+',maximum-scale='+scale+',minimum-scale='+scale+',user-scalable=no');
	///////////////////////// viewport end //////////////////////////////////
    //横屏状态检测
    if (window.orientation === 90 || window.orientation === -90) {
        isLandscape = true;
    };
    ///////////////////// system font-size check start //////////////////////
    //试探字体大小，用于检测系统字体是否正常
    var setFz = '100px';
    //给head增加一个隐藏元素
    var headEle = document.getElementsByTagName('head')[0],
        spanEle = document.createElement('span');
        spanEle.style.fontSize = setFz;
        spanEle.style.display = 'none';
        headEle.appendChild(spanEle);
    //判断元素真实的字体大小是否setFz
    //如果不相等则获取真实的字体换算比例
    var realFz = getComputedStyle(headEle).getPropertyValue('font-size');
    if(setFz !== 'realFz'){
        //去掉单位px，下面要参与计算
        setFz = parseFloat(setFz);
        realFz = parseFloat(realFz);
        //获取字体换算比例
        fontSizeRadio = setFz / realFz;
    };
    ///////////////////// system font-size check end //////////////////////
    var setBaseFontSize = function(){
        var deviceWidth = docEle.clientWidth,
            deviceHeight= docEle.clientHeight;
        if(isLandscape){
            deviceWidth = deviceHeight;
        };
        docEle.style.fontSize = deviceWidth * fontSizeRadio + 'px';
    };
    setBaseFontSize();
    //页面发生变化时重置font-size
    //防止多个事件重复执行，增加延迟300ms操作(防抖)
    var tid;
    window.addEventListener('resize', function() {
        clearTimeout(tid);
        tid = setTimeout(setBaseFontSize, 300);
    }, false);
    window.addEventListener('pageshow', function(e) {
        if (e.persisted) {
            clearTimeout(tid);
            tid = setTimeout(setBaseFontSize, 300);
        };
    }, false);
})(window, document);
```
* sass
```
//设计稿尺寸大小，假如设计稿宽度750
$baseDesignWidth = 750;
@function calc($val){
    @return $val / $baseDesignWidth;
}
//适配元素采用rem，假如设计稿中元素宽度180
.logo{
	width : calc(180rem);
}
//边框采用px，假如设计稿边框宽度1px
.box{
	border : 1px solid #ddd;
}
```

## 水平垂直居中
### 水平居中
* 块级元素使用margin: 0 auto;可以在父元素的中间位置居中，不过要记得设置块级元素的宽高。
* margin-left + border-left-width + padding-left + width + padding-rigth + border-right-width + margin-righ = width

### 垂直居中
* div使用绝对布局，设置margin:auto;并设置top、left、right、bottom的值相等即可，不一定要都是0
* top + margin-top + border-top-width + padding-top + height + padding-bottom + border-bottom-width + margin-bottom + bottom = height

### 水平垂直居中
* 使用绝对定位+负margin，这种方法需要知道子元素的高度
```
<style>
    .father {
        position: relative;
        width: 500px;
        height: 500px;
        background-color: blue;
    }
    .center {
        width: 100px;
        height: 100px;
        background-color: red;
        position: absolute;
        top: 50%;
        margin-top: -50px;
    }
</style>
<body>
    <div class="father">
        <div class="center">
            <span style="line-height: 100px">hello world</span>
        </div>
    </div>
</body>
```
* 相对定位+transform，这种情况不需要知道子元素的高度
```
<style>
    .father {
        position: relative;
        height: 500px;
        width: 500px;
        background-color: blue;
    }
    .center {
        position: relative;
        top: 50%;
        width: 200px;
        height: 200px;
        background-color: red;
        transform: translateY(-50%);
    }
</style>
<body>
    <div class="father">
        <div class="center">hello</div>
    </div>
</body>
```
* 设置vertical-align:middle让图片和文字居中显示
```
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
        .center {
            vertical-align: middle;
        }
    </style>
</head>
<body>
    <img class="center" src="buct.jpg" alt="" width="100px" height="100px"><span>hello world</span>
</body>
</html>
```
* flexbox
```
.warp {
  display: -webkit-flex;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-box;
  display: flex;
  -webkit-box-align: center;
  -moz-box-align: center;
  -ms-flex-align: center;
  -webkit-align-items: center;
  align-items: center;
  -webkit-box-pack: center;
  -moz-box-pack: center;
  -ms-flex-pack: center;
  -webkit-justify-content: center;
  justify-content: center;
  width: 200px;
  height: 200px;
  background-color: orange;
}
.example3 {
  width: 100px;
  height: 100px;
  background-color: red;
}
```
