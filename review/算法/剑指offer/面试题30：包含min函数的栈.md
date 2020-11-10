思想：

用数据栈+辅助栈。

辅助栈栈顶的元素始终是最小元素。

```javascript
var MinStack = function() {
    this.masterStack = [];
    this.otherStack = [];
};

MinStack.prototype.push = function(x) {
    this.masterStack.push(x);
    let {length} = this.otherStack;
    let top = this.otherStack[length-1];
    if(!length||x<top){
        this.otherStack.push(x);
    }else{
        this.otherStack.push(top);
    }
};

MinStack.prototype.pop = function() {
    this.masterStack.pop();
    this.otherStack.pop();
};

MinStack.prototype.top = function() {
    return this.masterStack[this.masterStack.length-1]
};

MinStack.prototype.min = function() {
    return this.otherStack[this.otherStack.length-1];
};
```

