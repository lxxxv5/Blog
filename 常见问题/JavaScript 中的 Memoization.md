# JavaScript 中的 Memoization

有些纯函数在运行时特别耗时，如果能维护一个参数到值的映射关系，在纯函数运行第一次时，将结果缓存起来，等到下一次以同样的参数运行时，将缓存的值直接返回即可。Memoization 就是这样的一种技术。

例如递归计算斐波那契数列：

```js
function fibonacci(num) {
  return num === 0 || num === 1 ? 1 : fibonacci(num - 1) + fibonacci(num - 2)
}
```

这里特意使用了无优化递归形式，能让计算过程更耗时，如下两次计算第 41 位，每次耗时约两秒左右：

```js
var date = new Date()
fibonacci(40)
console.log(new Date() - date) // 2286
fibonacci(40)
console.log(new Date() - date) // 4691
```

为了使第二次计算取得返回值更快，可以将第一次的参数和计算结果缓存起来：

```js
function fibonacci(num) {
  if (fibonacci.map.has(num)) return fibonacci.map.get(num)
  const result = fibonacci(num - 1) + fibonacci(num - 2)
  if (!fibonacci.map.has(num)) fibonacci.map.set(num, result)
  return result
}
fibonacci.map = new Map()
fibonacci.map.set(0, 1)
fibonacci.map.set(1, 1)
```

重新执行两次计算，会发现计算时间大大降低：

```js
var date = new Date()
fibonacci(40)
console.log(new Date() - date) // 2
fibonacci(40)
console.log(new Date() - date) // 5
```

将整个 Memoization 功能独立出来，写成一个高阶函数：

```js
const cache = Symbol()
function memoized(fn) {
  fn[cache] = {}
  return function (key) {
    if (fn[cache][key]) return fn[cache][key]
    const result = fn(key)
    fn[cache][key] = result
    return result
  }
}
```

使用示例如下：

```js
var memoFibonacci = memoized(fibonacci)
```

## 总结

函数记忆（memoization）将函数的参数和结果值，保存在对象当中，用一部分的内存开销来提高程序计算的性能，常用在递归和重复运算较多的场景。

需要注意的是，被处理的函数必须是纯函数，否则由同样的入参得不到相同的结果的话，缓存结果就没意义了。
