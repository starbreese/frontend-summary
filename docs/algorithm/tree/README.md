## 通用树
### 找到两个dom节点的最近公共祖先节点
```
function commonParentNode(oNode1, oNode2) {
    var parents1 = [],
    parents2 = [];
    var pNode1 = oNode1,
    pNode2 = oNode2;
    while(oNode1.parentNode){
        parents1.push(oNode1.parentNode);
        oNode1 = oNode1.parentNode;
    }
    while(oNode2.parentNode){
        parents2.push(oNode2.parentNode);
        oNode2 = oNode2.parentNode;
    }
    var len1 = parents1.length,
    len2 = parents2.length;
    //确保pNode1存放更小的元素，因为len越小越接近于body
    if(len1<len2){
        var temNode = pNode1;
        pNode1 = pNode2;
        pNode2 = temNode;
    }
    while(pNode1.parentNode !== pNode2.parentNode && pNode1.parentNode !== pNode2){
        pNode1 = pNode1.parentNode;
    }
    return pNode1.parentNode;
}
```
## 二叉树
## B+树
## 红黑树
