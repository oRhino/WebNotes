
## promise的三种状态

- pendding(待定) 初始状态,既没有兑现,也没有拒绝
- fulfilled(已兑现) 意味着操作成功
- rejected(已拒绝) 意味着操作失败

状态是单向的,只能pendding->fulfilled,或者pendding->rejected.


## 静态方法
- all(ES2015) 全部Promise执行成功,或者任意一个执行失败.
- allSettled(ES2020) 执行多个Promise,不论成功和失败,结果全部返回.
- any(ES2021) 接受一个Promise集合,返回第一个成功者.
- race(ES2015) Promise集合中,返回最快的Promise触发结果.
- resolve 返回一个解析过参数的Promise对象.
- reject 返回一个状态为失败的Promise对象.


## 原型方法
- then 返回一个Promise,最多两个参数,成功和失败后的回调函数.
- catch 返回一个Promise,并处理被拒绝的情况.
- finally 返回一个Promise,在Promise结束时,无论成功或者失败都执行该回调.

## 应用
1. 延迟函数

缺点: 
1. 无法获取返回值
2. 添加后续操作困难
```
function delay(fn, delay, context) {
    let defaultDelay = delay || 5000;
    let ticket;
    return {
        run(...args) {
            ticket = setTimeout(async () => {
                fn.apply(context, args);
            }, defaultDelay)
        },
        cancel: () => {
            clearTimeout(ticket);
        }
    }
}
const { run, cancel } = delay(() => { console.log("111"); return 2; }, 3000);
run();
setTimeout(() => {
    cancel();
}, 1000);
```
使用Promise

```
function isFunction(fn) {
    return typeof fn === 'function' || fn instanceof Function
}

function delay(fn, delay, context) {
    let defaultDelay = delay || 5000;
    if (!isFunction(fn)) {
        return {
            run: () => Promise.resolve(),
            cancel: noop
        }
    }
    let ticket;
    let executed = false;
    return {
        run(...args) {
            return new Promise((resolve, reject) => {
                if (executed === true) {
                    // delay function has been executed
                    return;
                }
                executed = true;
                ticket = setTimeout(async () => {
                    try {
                        const res = await fn.apply(context, args);
                        resolve(res);
                    } catch (err) {
                        reject(err)
                    } finally {
                        clearTimeout(ticket);
                    }
                }, defaultDelay)
            })
        },
        cancel: () => {
            clearTimeout(ticket);
        }
    }
}

//测试
const { run, cancel } = delay(() => { return "函数执行结果" }, 3000);

run().then((result) => {
    console.log("result:", result);
})

run().then(() => {
    console.log("多次调用run result:", result);
});
```

2. 重复执行
