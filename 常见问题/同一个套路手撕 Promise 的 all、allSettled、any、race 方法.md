# 同一个套路手撕 Promise 的 all、allSettled、any、race 方法

## 异同点

先来看看他们的共同点：

1. 这些方法的参数都接收一个 promise 的 iterable 类型（注：Array，Map，Set 都属于 ES6 的 iterable 类型）的输入。
2. 这些方法都返回一个 `Promise` 实例。

再看看他们的不同点：

1. 返回的 Promise 实例的状态改变时机不同
   - all 方法在所有输入的 Promise 实例都 resolve 后执行自身的 resolve 回调，在任意一个输入的 Promise 实例 reject 后执行自身的 reject 回调。
   - allSettled 方法在所有输入的 Promise 实例都改变了状态（执行 resolve 回调或 reject 回调）后执行自身的 resolve 回调。
   - any 方法在所有输入的 Promise 实例都 reject 后执行自身的 reject 回调，在任意一个输入的 Promise 实例 resolve 后执行自身的 resolve 回调。
   - race 方法在任意一个输入的 Promise 实例改变状态后以相同的状态改变自身。
2. 返回的 Promise 实例的[终值（eventual）或拒因（reason）](https://www.ituring.com.cn/article/66566)不同
   - all 方法返回的 Promise 实例终值是一个数组，数组的成员是所有输入的 Promise 实例的终值，并将会按照参数内的 promise 顺序排列，而不是由调用 promise 的完成顺序决定。拒因是输入的 Promise 实例中第一个状态变为 rejected 的拒因。
   - allSettled 方法返回的 Promise 实例终值也是一个数组，顺序同 promise 输入顺序，其中每个成员在输入 promise 为 resolved 状态时为 `{status:'fulfilled', value:同一个终值}`，rejected 状态时为 `{status:'rejected', reason:同一个拒因}`。
3. 参数为空迭代对象时，返回值不同
   - all 方法会同步地返回一个已完成（resolved）状态的 promise，其终值是一个空数组。
   - allSettled 方法和 all 方法表现相同。
   - any 方法会同步地返回一个已失败（rejected）状态的 promise，其拒因是一个 `AggregateError` 对象。
   - race 方法返回一个永远等待的 promise。

以上是这四个 `all`、`allSettled`、`any`、`race` 方法的横向对比，如果想综合查看某个方法的描述可以翻阅文章末尾的参考资料。

接下来就是我们的实现环节，但为了简化，我们仅处理参数为数组的情况，其他 iterable 类型（例如 Set、Map、String 等）的参数差异主要在于取成员个数或遍历方式不一样。

首先处理的是当参数为空迭代对象时，我们的模板长这样：

```diff
+function template(promises){
+    if(promises.length === 0){
+       //  根据不同情况作处理
+    }
+}
```

当参数的长度不为空时，返回一个新的 Promise：

```diff
function template(promises){
    if(promises.length === 0){
       //  根据不同情况作处理
    }
+   return new Promise((resolve,reject) => {
+       //  根据不同情况处理
+   })
}
```

定义一个结果收集数组和一个表示符合条件的 promise 状态个数变量：

```diff
function template(promises){
    if(promises.length === 0){
       //  根据不同情况作处理
    }
+   let result = [], num = 0;
    return new Promise((resolve,reject) => {
        //  根据不同情况处理
    })
}
```

在返回的 promise 内部遍历参数，为其添加 then 回调，回调中根据不同情况作处理，最后的模板如下：

```js
function template(promises) {
  if (promises.length === 0) {
    //  根据不同情况作处理
  }
  let result = [],
    num = 0
  return new Promise((resolve, reject) => {
    const check = () => {
      if (num === promises.length) {
        //    根据不同情况调用 resolve 或 reject
      }
    }
    promises.forEach(item => {
      Promise.resolve(item).then(
        res => {
          //  根据不同情况处理 result、num 和调用 resolve、reject、check 方法
        },
        err => {
          //  根据不同情况处理 result、num 和调用 resolve、reject、check 方法
        }
      )
    })
  })
}
```

插播一下 Promise.resolve 这个函数：

> Promise.resolve(value)方法返回一个以给定值解析后的 Promise 对象。如果这个值是一个 promise ，那么将返回这个 promise ；如果这个值是 thenable（即带有"then" 方法），返回的 promise 会“跟随”这个 thenable 的对象，采用它的最终状态；否则返回的 promise 将以此值完成。

因为 promises 的成员里可能混入了一些不是 promise 的值，所以用 Promise.resolve 去解析后就能统一为其添加 then 回调了。

## 具体实现

1. all 方法

   ```js
   function all(promises) {
     if (promises.length === 0) return Promise.resolve([])
     return new Promise((resolve, reject) => {
       let result = []
       let num = 0
       const check = () => {
         if (num === promises.length) {
           resolve(result)
         }
       }
       promises.forEach((item, index) => {
         Promise.resolve(item).then(
           res => {
             result[index] = res
             num++
             check()
           },
           err => {
             reject(err)
           }
         )
       })
     })
   }
   ```

2. allSettled 方法

   ```js
   function allSettled(promises) {
     if (promises.length === 0) return Promise.resolve([])
     return new Promise((resolve, reject) => {
       let result = []
       let num = 0
       const check = () => {
         if (num === promises.length) resolve(result)
       }
       promises.forEach((item, index) => {
         Promise.resolve(item).then(
           res => {
             result[index] = { status: 'fulfilled', value: res }
             num++
             check()
           },
           err => {
             result[index] = { status: 'rejected', reason: err }
             num++
             check()
           }
         )
       })
     })
   }
   ```

3. any 方法

   ```js
   function any(promises) {
     if (promises.length === 0) {
       reject(new AggregateError('No Promise in Promise.any was resolved'))
     }
     return new Promise((resolve, reject) => {
       let result = []
       let num = 0
       const check = () => {
         if (num === promises.length) {
           reject(new AggregateError('No Promise in Promise.any was resolved'))
         }
       }
       promises.forEach((item, index) => {
         Promise.resolve(item).then(
           res => {
             resolve(res)
           },
           err => {
             result[index] = err
             num++
             check()
           }
         )
       })
     })
   }
   ```

4. race 方法，相对于其他三个方法，race 方法实现比模版更简单点

   ```js
   function race(promises) {
     if (promises.length === 0) return Promise.resolve()
     return new Promise((resolve, reject) => {
       promises.forEach(item => {
         item.then(resolve, reject)
       })
     })
   }
   ```

## 参考资料

1. [MDN Promise.all()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)
2. [MDN Promise.allSettled()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled)
3. [MDN Promise.any()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/any)
4. [MDN Promise.race()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/race)
5. [MDN Promise.resolve()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve)
