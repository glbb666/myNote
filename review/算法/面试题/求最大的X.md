题目：

对于任意子序列，我们可以求一个`X`
`X=sum(subArray) * min(subArray)`
求最大的`X`，要求时间复杂度`o(n)`

例如：`[3,1,6,4,5,2]`

我们对`4`求的`X`为：`60 = (6+4+5)*4`   

我们其实可以把求值过程分为两步：

- `sum(start, end) => o(n) -> o(1)`

- 求每个数作为最小值时的start, end
  stack => 确保top到bottom顺序是从大到小

第一步：前缀和

把求和从`O(n)`简化为`O(1)`，思路是用一个数组存储**前缀和**

> 前缀和：数组的某项下标之前（包括此项元素）的所有元素的和。
>
> 设b[]为前缀和数组，a[]为原数组，根据这句话可以得到前缀和的定义式：
>
> ![b[i]=\sum_{j=0}^{i}a[j]](images/gif-1604414352356.gif)

根据上面的定义，前`i`个数的和 `sum[i] = sum[i-1] + a[i]` 

且根据上述表达式我们可以以`O(1)`求出区间`[i,j]`的区间和     

![sum[i,j]=b[j]-b[i-1]](images/gif.gif)

第二步：最小栈

我们可以根据剑指offer第30题学习最小栈的内容



