## 应用场景
react 高阶组件
### 定义
### when
* 

### after
1. react高阶组件
```
import React from 'react';
const yellowHOC = WrapperComponent => {
  return class extends React.Component {
    render() {
      <div style={{ backgroundColor: 'yellow' }}>
        <WrapperComponent {...this.props} />
      </div>;
    }
  };
};
export default yellowHOC;
```

使用其来装饰目标组件
```
import React from 'react';
import yellowHOC from './yellowHOC';

class TargetComponent extends Reac.Compoment {
  render() {
    return <div>66666</div>;
  }
}

export default yellowHOC(TargetComponent);

```

2. 又一个例子
```
const kuanWrite = function() {
  this.writeChinese = function() {
    console.log('我只会写中文');
  };
};
// 通过装饰器给阿宽加上写英文的能力
const Decorator = function(old) {
  this.oldWrite = old.writeChinese;
  this.writeEnglish = function() {
    console.log('给阿宽赋予写英文的能力');
  };
  this.newWrite = function() {
    this.oldWrite();
    this.writeEnglish();
  };
};
const oldKuanWrite = new kuanWrite();
const decorator = new Decorator(oldKuanWrite);
decorator.newWrite();
```