## 应用场景

### 定义
为了解决我们不兼容的问题，把一个类的接口换成我们想要的接口


### after
```
/** ---------------新的代码------------ */
const kuanLanguage = function() {
  this.japaneseLanguage = function() {
    console.log('阿宽写日文');
  };
};
const pengLookLanguage = function() {
  this.lookLanguage = function() {
    console.log('小彭只看得懂英文');
  };
};
// 通过适配器进行接口适配,从而满足我的需求
const AdapterLanguage = function(user) {
  this.language = user.lookLanguage;
};

AdapterLanguage.prototype = new kuanLanguage();
AdapterLanguage.prototype.japaneseLanguage = function(name) {
  console.log(`通过适配器, ${name}也能听日文了`);
};
const peng = new pengLookLanguage();
const adapterPeng = new AdapterLanguage(peng);
adapterPeng.japaneseLanguage('小彭');
```
