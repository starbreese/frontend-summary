## 常用方法
* splice
    * 通过删除或替换现有元素或者原地添加新的元素来修改数组,并以数组形式返回被修改的内容。**此方法会改变原数组**
    * array.splice(start[, deleteCount[, item1[, item2[, ...]]]])
        
        start​：指定修改的开始位置（从0计数）。如果超出了数组的长度，则从数组末尾开始添加内容；如果是负值，则表示从数组末位开始的第几位（从-1计数，这意味着-n是倒数第n个元素并且等价于array.length-n）；如果负数的绝对值大于数组的长度，则表示开始位置为第0位。

        deleteCount： 可选，整数，表示要移除的数组元素的个数。如果 deleteCount 大于 start 之后的元素的总数，则从 start 后面的元素都将被删除（含第 start 位）。如果 deleteCount 被省略了，或者它的值大于等于array.length - start(也就是说，如果它大于或者等于start之后的所有元素的数量)，那么start之后数组的所有元素都会被删除。如果 deleteCount 是 0 或者负数，则不移除元素。这种情况下，至少应添加一个新元素。

        item1, item2, ... 可选：要添加进数组的元素,从start 位置开始。如果不指定，则 splice() 将只删除数组元素。
    * 返回值<br/>
        由被删除的元素组成的一个数组。如果只删除了一个元素，则返回只包含一个元素的数组。如果没有删除元素，则返回空数组。
* slice
    * 返回一个新的数组对象，这一对象是一个由 begin 和 end 决定的原数组的浅拷贝（包括 begin，不包括end）。**原始数组不会被改变**
    * arr.slice([begin[, end]]) **begin，end若为负数，则是逆序中的位置，取出的数组不改变原先的位置**

        begin 可选：**闭区间** 提取起始处的索引（从 0 开始），从该索引开始提取原数组元素。如果该参数为负数，则表示从原数组中的倒数第几个元素开始提取，slice(-2) 表示提取原数组中的倒数第二个元素到最后一个元素（包含最后一个元素）。如果省略 begin，则 slice 从索引 0 开始。如果 begin 大于原数组的长度，则会返回空数组。
        
        end 可选：**开区间** 提取终止处的索引（从 0 开始），在该索引处结束提取原数组元素。slice 会提取原数组中索引从 begin 到 end 的所有元素（包含 begin，但不包含 end）。slice(1,4) 会提取原数组中从第二个元素开始一直到第四个元素的所有元素 （索引为 1, 2, 3的元素）。如果该参数为负数， 则它表示在原数组中的倒数第几个元素结束抽取。 slice(-2,-1) 表示抽取了原数组中的倒数第二个元素到最后一个元素（不包含最后一个元素，也就是只有倒数第二个元素）。如果 end 被省略或大于数组长度，则slice 也会一直提取到原数组末尾。

## 数组去重
1. es6
```
function unique (arr) {
  return Array.from(new Set(arr))
}
```
**这样简写 [...new Set(arr)]**


2. for
```
function unique(arr){            
    for(var i=0; i<arr.length; i++){
        for(var j=i+1; j<arr.length; j++){
            //第一个等同于第二个，splice方法删除第二个
            if(arr[i]==arr[j]){
                arr.splice(j,1);
                j--;
            }
        }
    }
  return arr;
}
```

3. indexOf, includes
```
function unique(arr) {
    if (!Array.isArray(arr)) {
        console.log('type error!')
        return
    }
    var array = [];
    for (var i = 0; i < arr.length; i++) {
        if (array.indexOf(arr[i]) === -1) {    //NaN、{}没有去重
       // if(!array.includes( arr[i]) ) {      //{}没有去重
            array.push(arr[i])
        }
    }
    return array;
}
```

4. object的key, true, 'true'会有问题
```
function unique(arr) {
    if (!Array.isArray(arr)) {
        console.log('type error!')
        return
    }
    var arrry= [];
     var  obj = {};
    for (var i = 0; i < arr.length; i++) {
        if (!obj[arr[i]]) {
            arrry.push(arr[i])
            obj[arr[i]] = 1
        } else {
            obj[arr[i]]++
        }
    }
    return arrry;
}
    var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
    console.log(unique(arr))
//[1, "true", 15, false, undefined, null, NaN, 0, "a", {…}]   
```

5. hasOwnProperty, typeof区分类型，生成唯一key,解决4中的问题
```
function unique(arr) {
    var obj = {};
    return arr.filter(function(item, index, arr){
        return obj.hasOwnProperty(typeof item + item) ? false : (obj[typeof item + item] = true)
    })
}
    var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
        console.log(unique(arr))
//[1, "true", true, 15, false, undefined, null, NaN, "NaN", 0, "a", {…}]
```

6. map
```
function arrayNonRepeatfy(arr) {
    let map = new Map();
    let array = new Array();  // 数组用于返回结果
    for (let i = 0; i < arr.length; i++) {
        if(map.has(arr[i])) {  // 如果有该key值
            map.set(arr[i], true); 
        } else { 
            map.set(arr[i], false);   // 如果没有该key值
        array.push(arr[i]);
        }
    } 
    return array ;
}
var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
console.log(unique(arr))
//[1, "a", "true", true, 15, false, 1, {…}, null, NaN, NaN, "NaN", 0, "a", {…}, undefined]
```

7. reduce,includes
```
function unique(arr){
    return arr.reduce((prev,cur) => prev.includes(cur) ? prev : [...prev,cur],[]);
}
var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
console.log(unique(arr));
// [1, "true", true, 15, false, undefined, null, NaN, "NaN", 0, "a", {…}, {…}]
```

8. filter
```
function unique(arr) {
    return arr.filter(function(item, index, arr) {
        //当前元素，在原始数组中的第一个索引==当前索引值，否则返回当前元素
        return arr.indexOf(item, 0) === index;
    });
}
var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
console.log(unique(arr))
//[1, "true", true, 15, false, undefined, null, "NaN", 0, "a", {…}, {…}]
```

9. 递归
```
function unique(arr) {
    var array= arr;
    var len = array.length;
    array.sort(function(a,b){   //排序后更加方便去重
        return a - b;
    })
    function loop(index){
        if(index >= 1){
            if(array[index] === array[index-1]){
                array.splice(index,1);
            }
            loop(index - 1);    //递归loop，然后数组去重
        }
    }
    loop(len-1);
    return array;
}
var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
console.log(unique(arr))
//[1, "a", "true", true, 15, false, 1, {…}, null, NaN, NaN, "NaN", 0, "a", {…}, undefined]
```