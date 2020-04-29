## 应用场景
### 定义
要实现某一个功能，有多种方案可以选择。我们定义策略，把它们一个个封装起来，并且使它们可以相互转换。
### when
* 各判断条件下的策略相互独立且可复用
* 策略内部逻辑相对复杂
* 策略需要灵活组合

### before
```
function checkAuth(data) {
    if (data.role !== 'juejin') {
      console.log('不是掘金用户');
      return false;
    }
    if (data.grade < 1) {
      console.log('掘金等级小于 1 级');
      return false;
    }
    if (data.job !== 'FE') {
      console.log('不是前端开发');
      return false;
    }
    if (data.type !== 'eat melons') {
      console.log('不是吃瓜群众');
      return false;
    }
}
```

### after
* 封装不同方案
```
// 维护权限列表
const jobList = ['FE', 'BE'];
// 策略
var strategies = {
  checkRole: function(value) {
    if (value === 'juejin') {
      return true;
    }
    return false;
  },
  checkGrade: function(value) {
    if (value >= 1) {
      return true;
    }
    return false;
  },
  checkJob: function(value) {
    if (jobList.indexOf(value) > 1) {
      return true;
    }
    return false;
  },
  checkEatType: function(value) {
    if (value === 'eat melons') {
      return true;
    }
    return false;
  }
};
```
* 管理策略
```
// 校验规则
var Validator = function() {
  this.cache = [];

  // 添加策略事件
  this.add = function(value, method) {
    this.cache.push(function() {
      return strategies[method](value);
    });
  };

  // 检查
  this.check = function() {
    for (let i = 0; i < this.cache.length; i++) {
      let valiFn = this.cache[i];
      var data = valiFn(); // 开始检查
      if (!data) {
        return false;
      }
    }
    return true;
  };
};
```
* 应用
```
// 小彭使用策略模式进行操作
var compose1 = function() {
    var validator = new Validator();
    const data1 = {
      role: 'juejin',
      grade: 3
    };
    validator.add(data1.role, 'checkRole');
    validator.add(data1.grade, 'checkGrade');
    const result = validator.check();
    return result;
};
```
