### 是什么

浏览器提供的一个用于高效实现动画的 API。

### 执行时机

它会在**浏览器下一次重绘之前**被调用。

在一帧的渲染流水线中，大致顺序是：

1. 执行宏任务和微任务
2. 执行所有 `requestAnimationFrame` 回调（ **这是 JavaScript 在渲染前最后一次修改 DOM/样式的机会** ）
3. 样式计算
4. 布局
5. 绘制
6. 合成

因此，`requestAnimationFrame` 的回调 **紧挨着渲染发生** ，保证你对 DOM 或样式的修改能在同一帧中呈现，不会额外延迟。

### 为什么需要 `requestAnimationFrame`

通过定时器 `setTimeout` 或 `setInterval` 实现 `js`动画存在两个问题，第一个是动画的**循环时间没有统一的标准**，设置长了动画显得不够平滑，设置短了多次重绘频率会达到瓶颈；第二个问题是**定时器是一个宏任务，第二个时间参数只是指定了多久后将动画添加到任务队列中，没有办法保证动画的立刻执行**。为了解决这些问题，`H5` 中加入了 `requestAnimationFrame`

### 优点

- `requestAnimationFrame` 会把**每一帧中的所有 DOM 操作集中**起来，在一次重绘或回流中就完成，并且重绘或回流的时间间隔紧紧跟随**浏览器的刷新频率**（1秒60帧，每帧 `16.7ms`)
- 在隐藏或不可见的元素中，`requestAnimationFrame` 将不会进行重绘或回流，这当然就意味着更少的 `CPU`、`GPU `和内存使用量
- `requestAnimationFrame `是由浏览器专门为动画提供的 `API`，在运行时浏览器会自动优化方法的调用，并且如果页面不是激活状态下的话，动画会自动暂停，有效节省了 CPU 开销

### 应用

#### js动画

`requestAnimationFrame `本就是为动画而生，与定时器的用法非常相似，下面是一个例子，实现了点击方块开始旋转（每次点击都会重置动画，从当前角度继续递增），点击停止按钮停止旋转，并且不会出现多个动画叠加的问题。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>旋转方块 - 正确示例</title>
    <style>
        #box {
            width: 100px;
            height: 100px;
            background-color: cadetblue;
            margin: 20px;
            cursor: pointer;
        }
        #stop {
            margin: 20px;
            cursor: pointer;
        }
        .info {
            margin: 20px;
            font-family: monospace;
        }
    </style>
</head>
<body>
    <div id="box"></div>
    <button id="stop">停止旋转</button>
    <div class="info">点击方块开始/重新开始旋转（从当前角度继续），点击停止按钮暂停</div>

    <script>
        var deg = 0;                // 当前旋转角度
        var animationId = null;    // 当前动画的 requestAnimationFrame ID
        var box = document.getElementById("box");

        // 旋转函数：更新角度并递归调用自身
        function rotate() {
            // 更新旋转角度并应用到元素
            box.style.transform = 'rotate(' + deg + 'deg)';
            deg++;                  // 每次增加1度，实现连续旋转
            // 注册下一帧动画，并保存ID
            animationId = requestAnimationFrame(rotate);
        }

        // 停止动画的函数
        function stopRotate() {
            if (animationId !== null) {
                cancelAnimationFrame(animationId);
                animationId = null;   // 清除ID，表示动画已停止
            }
        }

        // 点击方块：先停止当前动画（如果有），再启动新动画
        box.addEventListener('click', function() {
            // 如果已经有动画在运行，先停止它
            if (animationId !== null) {
                cancelAnimationFrame(animationId);
                animationId = null;
            }
            // 从当前 deg 值开始继续旋转
            rotate();
        });

        // 点击停止按钮：停止动画
        document.getElementById('stop').onclick = function() {
            stopRotate();
        };
    </script>
</body>
</html>
```

#### 大数据渲染

在大数据渲染过程中，如果不进行一些性能策略处理，就会出现 UI 卡死现象。

有个场景，将后台返回的十万条记录插入到表格中，如果一次性在循环中生成 DOM 元素，会导致页面卡顿5s左右。

这时候我们就可以用 requestAnimationFrame + Fragment的方案进行分步渲染，确定最好的时间间隔，使得页面加载过程中很流畅。

```js
var total = 100000;
var size = 100;
var count = total / size;
var done = 0;
var ul = document.getElementById('list');

function addItems() {
    var li = null;
    var fg = document.createDocumentFragment();
    // DocumentFragments 是文档片段，不是主DOM树的一部分。
    // 将元素附加到文档片段，然后将文档片段附加到DOM树。在DOM树中，文档片段被其所有的子元素所代替。

    for (var i = 0; i < size; i++) {
        li = document.createElement('li');
        li.innerText = 'item ' + (done * size + i);
        fg.appendChild(li);
    }

    ul.appendChild(fg);
    done++;

    if (done < count) {
        requestAnimationFrame(addItems);
    }
};

requestAnimationFrame(addItems);
```
