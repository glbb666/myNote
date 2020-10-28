思想：用两个数字`small`和`big`分别表示序列的最小值和最大值。

如果从`small`到`big`的序列和大于`s`，则可以去掉较小的值，否则增大最大的值，由于`s`最少也是两个数的和，因此`small`得小于中位数。

```javascript
function pushArr(result,small,big){
    const arr = [];
    for(let i = small;i<=big;i++){
        arr.push(i);
    }
    result.push(arr);
}
var findContinuousSequence = function(target) {
    let small = 1;
    let big = 2;
    let middle = Math.floor((target+1)/2);
    let curSum = small+big;
    let result = [];
    while(small<middle){
        while(small<middle&&curSum>target){
            curSum-=small;
            small++;
        }
        if(curSum===target){
            pushArr(result,small,big);
            console.log(result)
        }
        big++;
        curSum+=big; 
    }
    return result;
};
```

