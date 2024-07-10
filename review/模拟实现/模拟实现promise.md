```js
class MyPromise {
    constructor(executor) {
        // 初始状态
        this.state = 'pending';
        this.value = undefined;
        this.reason = undefined;
        this.onFulfilledCallbacks = [];
        this.onRejectedCallbacks = [];

        const resolve = (value) => {
            if (this.state === 'pending') {
                this.state = 'fulfilled';
                this.value = value;
                this.onFulfilledCallbacks.forEach(fn => fn());
            }
        };

        const reject = (reason) => {
            if (this.state === 'pending') {
                this.state = 'rejected';
                this.reason = reason;
                this.onRejectedCallbacks.forEach(fn => fn());
            }
        };

        try {
            executor(resolve, reject);
        } catch (err) {
            reject(err);
        }
    }

    then(onFulfilled, onRejected) {
        // 如果成功的回调不存在，给一个默认函数，返回value
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
        // 如果失败的回调不存在，给一个默认函数，抛出reason
        onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason };

        if (this.state === 'fulfilled') {
            setTimeout(() => onFulfilled(this.value), 0);
        }

        if (this.state === 'rejected') {
            setTimeout(() => onRejected(this.reason), 0);
        }

        if (this.state === 'pending') {
            this.onFulfilledCallbacks.push(() => {
                setTimeout(() => onFulfilled(this.value), 0);
            });
            this.onRejectedCallbacks.push(() => {
                setTimeout(() => onRejected(this.reason), 0);
            });
        }
    }
}

```
