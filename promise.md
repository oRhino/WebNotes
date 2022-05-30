
## promise的三种状态

- pendding(待定) 初始状态,既没有兑现,也没有拒绝
- fulfilled(已兑现) 意味着操作成功
- rejected(已拒绝) 意味着操作失败

状态是单向的,只能pendding->fulfilled,或者pendding->rejected.


## 静态方法
- all(ES2015) 全部Promise执行成功,或者任意一个执行失败.

将多个Promise包裹在一起形成一个新的Promise;新的Promise状态由包裹的所有Promise共同决定:
当所有的Promise状态变成fulfilled状态时，新的Promise状态为fulfilled，并且会将所有Promise的返回值 组成一个数组;
当有一个Promise状态为reject时，新的Promise状态为reject，并且会将第一个reject的返回值作为参数;
 
```
let  p1 = new Promise((resolve,reject) => {
    setTimeout(() => {
        resolve('1');
    }, 1000);
})

let  p2 = new Promise((resolve,reject) => {
    setTimeout(() => {
        resolve('2');
    }, 2000);
})
let  p3 = new Promise((resolve,reject) => {
    setTimeout(() => {
        resolve('3');
    }, 2000);
})

Promise.all([p1,p2,p3]).then(res => {
    console.log(res);
}).catch(err => {
    console.log(err);
})
//[ '1', '2', '3' ]
```
- allSettled(ES2020) 执行多个Promise,不论成功和失败,结果全部返回.

all方法有一个缺陷:当有其中一个Promise变成reject状态时，新Promise就会立即变成对应的reject状态。那么对于resolved的，以及依然处于pending状态的Promise，我们是获取不到对应的结果的;
在ES11(ES2020)中，添加了新的API Promise.allSettled:
该方法会在所有的Promise都有结果(settled)，无论是fulfilled，还是reject时，才会有最终的状态; 并且这个Promise的结果一定是fulfilled的;

allSettled的结果是一个数组，数组中存放着每一个Promise的结果，并且是对应一个对象的; 这个对象中包含status状态，以及对应的value/reason值;
```
let  p1 = new Promise((resolve,reject) => {
    setTimeout(() => {
        resolve('1');
    }, 1000);
})

let  p2 = new Promise((resolve,reject) => {
    setTimeout(() => {
        resolve('2');
    }, 2000);
})
let  p3 = new Promise((resolve,reject) => {
    throw new Error('failed');
})

Promise.allSettled([p1,p2,p3]).then(res => {
    console.log(res);
}).catch(err => {
    console.log(err);
})
// [{ status: 'fulfilled', value: '1' },{ status: 'fulfilled', value: '2' },{status: 'rejected',reason: Error: failed }]
```
- any(ES2021) 接受一个Promise集合,返回第一个成功者.

any方法会等到一个fulfilled状态，才会决定新Promise的状态;
如果所有的Promise都是reject的，那么也会等到所有的Promise都变成rejected状态;
如果所有的Promise都是reject的，那么会报一个AggregateError的错误。

- race(ES2015) Promise集合中,返回最快的Promise触发结果.
 race是竞技、竞赛的意思，表示多个Promise相互竞争，谁先有结果，那么就使用谁的结果;

- resolve 返回一个解析过参数的Promise对象.
- reject 返回一个状态为失败的Promise对象.


## 原型方法
- then 返回一个Promise,最多两个参数,成功和失败后的回调函数.
  1. 同一个promise可以被多次调用then方法
```
const promise = new Promise((resolve, reject) => { resolve("11") })
  
promise.then(res => {
  console.log("res1:", res)
})
promise.then(res => {
  console.log("res2:", res)
})
///两个then方法都会得到调用
```
 2. then方法是可以链式调用的,因为它的返回值是Promise
```
//如果返回的是一个普通值(数值/字符串/普通对象/undefined), 那么这个普通的值被作为一个新的Promise的resolve值
const promise = new Promise((resolve, reject) => {
  resolve("1")
})
promise.then(res => {
  console.log(res);
  return "2"
}).then(res => {
  console.log(res);
})

// 1  2 

// 如果返回的是一个Promise
promise.then(res => {
  console.log(res);
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(2)
    }, 3000)
  })
}).then(res => {
  console.log(res)
})
// 1 2 

// 如果返回的是一个对象, 并且该对象实现了thenable
promise.then(res => {
  console.log(res)
  return {
    then: function(resolve, reject) {
      resolve(2)
    }
  }
}).then(res => {
  console.log(res)
})
// 1 2 
```

- catch 返回一个Promise,并处理被拒绝的情况.
- finally 返回一个Promise,在Promise结束时,无论成功或者失败都执行该回调.

## resolve参数
1. 普通的值或者对象 pending -> fulfilled
2. 传入一个Promise,那么当前的Promise的状态会由传入的Promise来决定,相当于状态进行了移交
3. 传入一个对象, 并且这个对象有实现then方法(并且这个对象是实现了thenable接口),那么也会执行该then方法, 并且又该then方法决定后续状态

```
//传入一个promise
const newPromise = new Promise((resolve, reject) => {
  reject("err message")
})
new Promise((resolve, reject) => {
  resolve(newPromise)
}).then(res => {
  console.log("res:", res)
}, err => {
  console.log("err:", err)
})
// err: err message

//传入一个thenable
new Promise((resolve, reject) => {
  const obj = {
    then: function(resolve, reject) {
      resolve("resolve message")
    }
  }
  resolve(obj)
}).then(res => {
  console.log("res:", res)
}, err => {
  console.log("err:", err)
})
//res: resolve message

```
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
