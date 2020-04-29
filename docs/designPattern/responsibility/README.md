## 定义
避免请求发送者与接收者耦合在一起，让多个对象都有可能接收请求，将这些对象连接成一条链，并且沿着这条链传递请求，直到有对象处理它为止

## example
1. 如图所示，我们申请设备之后，接下来要选择收货地址，然后选择责任人，而且必须是上一个成功，才能执行下一个~
* before
```
function applyDevice(data) {
  // 处理巴拉巴拉...
  let devices = {};
  let nextData = Object.assign({}, data, devices);
  // 执行选择收货地址
  selectAddress(nextData);
}
function selectAddress(data) {
  // 处理巴拉巴拉...
  let address = {};
  let nextData = Object.assign({}, data, address);
  // 执行选择责任人
  selectChecker(nextData);
}
function selectChecker(data) {
  // 处理巴拉巴拉...
  let checker = {};
  let nextData = Object.assign({}, data, checker);
  // 还有更多
}
```
* after
