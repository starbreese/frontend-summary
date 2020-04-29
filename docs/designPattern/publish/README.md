## 应用场景
需求 : 申请成功后，需要触发对应的订单、消息、审核模块对应逻辑
### 定义
发布-订阅是一种消息范式，消息的发布者，不会将消息直接发送给特定的订阅者，而是通过消息通道广播出去，然后呢，订阅者通过订阅获取到想要的消息。
### when
* 各模块相互独立
* 存在一对多的依赖关系,依赖模块不稳定、依赖关系不稳定

### 与观察者模式的区别
![logo](../../assets/pubwatch_comp.png)

### before
```
function applySuccess() {
    // 通知消息中心获取最新内容
    MessageCenter.fetch();
    // 更新订单信息
    Order.update();
    // 通知相关方审核
    Checker.alert();
}
```

### after
封装eventEmitter
```
const EventEmit = function() {
  this.events = {};
  this.on = function(name, cb) {
    if (this.events[name]) {
      this.events[name].push(cb);
    } else {
      this.events[name] = [cb];
    }
  };
  this.trigger = function(name, ...arg) {
    if (this.events[name]) {
      this.events[name].forEach(eventListener => {
        eventListener(...arg);
      });
    }
  };
};
```
业务代码改动
```
let event = new EventEmit();
event.trigger('success');

MessageCenter.fetch() {
  event.on('success', () => {
    console.log('更新消息中心');
  });
}
Order.update() {
  event.on('success', () => {
    console.log('更新订单信息');
  });
}
Checker.alert() {
  event.on('success', () => {
    console.log('通知管理员');
  });
}
```