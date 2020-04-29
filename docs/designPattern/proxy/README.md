## 应用场景
代理模式是为其它对象提供一种代理以控制这个对象的访问，具体执行的功能还是这个对象本身
## when
* 模块职责单一且可复用
* 两个模块间的交互需要一定限制关系

## example
1. 邮件
```
// 发邮件，不是qq邮箱的拦截
const emailList = ['qq.com', '163.com', 'gmail.com'];
// 代理
const ProxyEmail = function(email) {
  if (emailList.includes(email)) {
    // 屏蔽处理
  } else {
    // 转发，进行发邮件
    SendEmail.call(this, email);
  }
};
const SendEmail = function(email) {
  // 发送邮件
};
// 外部调用代理
ProxyEmail('cvte.com');
ProxyEmail('ojbk.com');
```

2. 《JavaScript 设计模式与开发实践》
```
// 本体
var domImage = (function() {
  var imgEle = document.createElement('img');
  document.body.appendChild(imgEle);
  return {
    setSrc: function(src) {
      imgEle.src = src;
    }
  };
})();
// 代理
var proxyImage = (function() {
  var img = new Image();
  img.onload = function() {
    domImage.setSrc(this.src); // 图片加载完设置真实图片src
  };
  return {
    setSrc: function(src) {
      domImage.setSrc('./loading.gif'); // 预先设置图片src为loading图
      img.src = src;
    }
  };
})();
// 外部调用
proxyImage.setSrc('./product.png');
```
