---
title: 详解Promise
date: 2021-12-29 18:16:59
permalink: /pages/211ef2/
categories: 
  - 开发者手册
  - JavaScript
tags: 
  - 
---

> 本文参考阮一峰 《ES6教程》 [Promise 章节](https://es6.ruanyifeng.com/#docs/promise)编写，仅是作者学习阮大神的写作思路和方法，并无抄袭照搬之意。

## Promise 的含义

`Promise` 是异步编程的一种解决方案，比传统的解决方案 *回调函数* 更合理也更强大。它是由社区最早提出并实现的，后来 ES6 将它列入了语法标准，原生提供了 `Promise` 对象。

所谓 `Promise`，就是一个容器，里面保存着未来才会结束的事件的结果。

`Promise` 有以下两个特点：

- **对象的状态不受外界影响。**`Promise` 对象代表一个异步操作，它有三个状态：`pending`（进行中）、`fulfilled`（已成功） 和 `rejected`（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。

- **一旦状态改变，就不会再变，任何时候都可以得到这个结果。**`Promise` 状态的改变，只有两种可能，从 `pending` 到 `fulfilled`，从 `pending` 到 `rejected`。只要这两种状态发生了，它的状态就定型了，我们称它为 resolved。如果异步操作已经完成了，我们再给 `Promise` 添加回调，也能立即得到异步操作的结果。这与浏览器事件不同，浏览器事件如果发生了我们再去监听它，就永远都不会拿到结果了。

`Promise` 也有一些缺点：

- 首先，无法取消 `Promise`，一旦创建就立即执行了，无法中途取消；
- 其次，如果不设置回调函数，`Promise`内部抛出的错误不会反应到外部；
- 当处于 `pending` 状态时，无法得知当前进展到哪一个阶段（刚刚开始还是即将完成）。

## 基础用法

#### 创建实例

`Promise` 对象是一个构造函数，用来生成 `Promise` 实例。

下面代码创建一个 `Promise` 实例。

``` js
const promise = new Promise((resolve, reject) => {
  // Some Code ...
  if (/* 异步操作成功 */) {
    resolve(value)
  } else {
    reject(error)
  }
})
```

`Promise` 构造函数接受一个函数作为参数。这个函数的两个参数分别是 `resolve` 和 `reject`，它们也是函数，在异步操作完成后（成功或失败）调用，成功调用 `resolve`，失败调用 `reject`。

`Promise` 实例生成后，可以使用 `then` 和 `catch` 分别指定 `resolve` 和 `reject` 的回调函数。

``` js
promise
  .then(value => {
    // success
    console.log(value)
  })
  .catch(error => {
    // failure
    console.error(error)
  })
```

这两个回调函数都是可选的，我们不一定要提供，这就是上文提到的，如果我们不写 `catch` 回调，就无法捕获到 `Promise` 实例内部抛出的错误了。

::: tip
也可以在 `then` 参数里定义第二个回调函数来取代 `catch`，但这样 `catch` 就无法捕获 `then` 中的错误了，因此**建议总是使用 `promise.then().catch()`**。

``` js
Promise.resolve()
  .then(value => {
    // success
    throw new Error('find a error')
  }, error => {
    // failure
    console.error(error)
  })
```

无法捕获到第一个回调中的错误。
:::

#### 立即执行

`Promise` 实例创建后就会立即执行。

``` js
new Promise((resolve) => {
  console.log(1)
  setTimeout(() => {
    console.log(3)
    resolve()
  }, 1000)
})

console.log(2)
```

上面的示例会依次打印 1，2，3。

这里表现跟 `setTimeout(() => {}, 0)` 不一样，涉及到事件循环（Event Loop）和宏（微）任务，请自行百度。

::: warning

调用 `resolve` 或 `reject` 后，后面的代码还会继续执行。

我们稍微改一下上面的例子：

``` js {6}
new Promise((resolve) => {
  console.log(1)
  setTimeout(() => {
    console.log(3)
    resolve()
    console.log(4)
  }, 1000)
})
  .then(res => {
    console.log(5)
  })

console.log(2)
```

上面例子的结果是：1，2，3，4，5。

因为 `resolve()` 调用后，后面的代码还是会继续执行。而改变了状态的 `Promise` 是在本轮事件循环的末尾执行，总是晚于本轮循环的同步任务。

一般来说，调用 `resolve()` 或 `reject()` 以后，`Promise` 的使命就完成了，后继操作应该放到 then 方法里面，而不应该直接写在 resolve 或 reject 的后面。所以，最好在它们前面加上 return 语句，这样就不会有意外。
:::

#### 链式调用

`Promise` 实例也支持链式调用。

`then` 回调返回的也是一个 `Promise` 实例，所以它后面可以继续调用 `then`，从而实现链式调用。

``` js {4}
getUserInfo()
  .then(userInfo => {
    // ...
    return getFundList(userInfo.userId)
  })
  .then(fundList => {
    console.log(fundList)
  })
```

上面代码中，第一个 then 方法指定的回调函数，返回的是另一个 `Promise` 对象。这时，第二个 then 方法指定的回调函数，就会等待这个新的 `Promise` 对象状态发生变化。并且会将返回结果作为参数，传入第二个回调函数。

如果第一个 then 没有返回值，第二个 then 回调的参数就是 `undefined`，并且会立即执行。

``` js
const promise = (ms) => {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve(ms)
    }, ms)
  })
}

promise(3000)
  .then(res => {
    // 1
    console.log(res)
    promise(5000)
  })
  .then(res => {
    // 2
    console.log(res)
  })
```

打印结果：等待 3 秒后，打印 3000 和 undefined。

如果两个异步请求是并发的，我们需要使用 `Promise.all()`[:arrow_down:](#promise-all)。

## API

#### Promise.resolve()

有时候需要将现有对象转为 `Promise` 对象，`Promise.resolve()` 就起到这个作用。

``` js
const p = Promise.resolve({name: 'jimdeng'}) // Promise {<fulfilled>: {...}}

p.then(res => { console.log(res) }) // {name: 'jimdeng'}
```

上面的 `Promise.resolve()` 等价于下面的写法：

``` js
const p = new Promise(resolve => resolve({name: 'jimdeng'}))
```

#### Promise.reject()

`Promise.reject()` 方法返回一个新的 `Promise` 实例，它的状态是 rejected。

``` js
const p = Promise.reject(new Error('出错了'))
// 等价于
Promise((resolve, reject) => reject(new Error('出错了')))

p.catch(error => {
  console.log(error.message)
})
```

#### Promise.all()

`Promise.all()` 方法将多个 `Promise` 实例，包装成一个新的 `Promise` 实例。

``` js
const p = Promise.all([p1, p2, p3])
```

上面代码中，`Promise.all()` 方法接受一个数组作为参数，`p1，p2，p3` 都是 `Promise` 实例，如果不是，就会先调用 `Promise.resolve()` 将参数转为 `Promise` 实例，再进一步处理。

`p` 的状态由 `p1，p2，p3` 决定，分为两种情况：

(1) **只有** `p1，p2，p3` 的状态都变成 fulfilled，`p` 的状态才会变成 fulfilled。此时 `p1，p2，p3` 的返回值组成一个数组，传递给 `p` 的回调函数。

(2) **只要** `p1，p2，p3` 之中有一个被 rejected，`p` 的状态就会变成 rejected。此时第一个被 rejected 的实例的返回值，会传递给 `p` 的回调函数。

看一个具体的例子：

``` js
const p = Promise.all([
  Promise.resolve(1),
  2,
  {num: 3}
])

p.then(([res1, res2, res3]) => {
  console.log(res1, res2, res3) // 1, 2, {num: 3}
})
```

::: warning
需要注意的是，作为参数的 `Promise` 实例，如果自己已经定义了 catch 方法，那么它一旦被 rejected，就不会触发 `Promise.all()` 的 catch 方法了。

同样，作为参数的 `Promise` 实例，如果自己已经定义了 then 方法，那么在 `Promise.all()` 的 then 回调参数中，值为 undefined。

**因此，建议在使用 `Promise.all()`，和后续介绍的 `Promise.race()` 等方法时，不要在参数 `Promise` 实例后添加回调，而总在最后处理。**

``` js {5,6,7,13,14,15}
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(new Error('p1'))
  }, 1000)
}).catch(error => {
  console.error('p1', error.message)
})

const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(new Error('p2'))
  }, 2000)
}).catch(error => {
  console.error('p2', error.message)
})

Promise.all([p1, p2])
  .catch(error => {
    console.error('Promise.all', error.message)
  })

```

结果打印："p1" "p1" 、"p2" "p2"，不会执行 `Promise.all()` 后的 catch 回调。

如果我们取消 `p1` 后的 catch 回调，由于 `p2` 的 catch 回调要 2 秒后才执行，而 `Promise.all()` 只要有一个参数 rejected，就会进入 catch，`p1` 在 1 秒后就 rejected 了。

所以代码执行后 1 秒后会打印 "Promise.all" "p1"，2 秒后还是会打印 "p2" "p2"。
:::


#### Promise.race()

`Promise.race()` 方法同样是将多个 `Promise` 实例，包装成一个新的 `Promise` 实例。

``` js
const p = Promise.race([p1, p2, p3]);
```

上面代码中，只要 `p1、p2、p3` 之中有一个实例率先改变状态，`p` 的状态就跟着改变。那个率先改变的 `Promise` 实例的返回值，就传递给 `p` 的回调函数。

``` js
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    reject(new Error('p1'))
  }, 1000)
})

const p2 = new Promise((resolve) => {
  setTimeout(() => {
    resolve('p2')
  }, 2000)
})

Promise.race([p1, p2])
  .then(res => {
    console.log('Promise.all--fulfilled', res)
  })
  .catch(error => {
    console.error('Promise.all--rejected', error.message)
  })
```

结果打印："Promise.all--rejected" "p1"。

如果分别在 `p1，p2` 后续添加回调，执行顺序就会是 `p1` 的 catch，`p2` 的 then，`Promsie.race()` 的 then。

这样的结果难免有些混乱。**因此，再次声明，只在最后添加回调**。

#### Promise.allSettled()

`Promise.allSettled()` 方法同样接受一个数组作为参数。只有等到参数数组的所有 `Promise` 对象都发生状态变更（不管是 fulfilled 还是 rejected ），返回的 `Promise` 对象才会发生状态变更。

返回的 `Promise` 对象一旦发生状态变更，状态总是 fulfilled，不会变成 rejected。而且它的回调函数接受一个数组作为参数，该数组包含原始 `promises` 集合中每个 `promise` 的结果。

``` js
const resolved = Promise.resolve(42);
const rejected = Promise.reject(-1);

const allSettledPromise = Promise.allSettled([resolved, rejected]);

allSettledPromise.then(function (results) {
  console.log(results);
});
// [
//   { status: 'fulfilled', value: 42 },
//   { status: 'rejected', reason: -1 }
// ]
```

`Promise.allSetteld()` 是在 ES2020 才引入的，下面的它的兼容性表格。

![](https://cdn.jsdelivr.net/gh/jimdeng92/static_1@main/微信截图_20211230183655.6ig92rjdn9c0.webp)

#### Promise.any()

ES2021 引入了 `Promise.any()` 方法。

只要参数实例有一个变成 fulfilled 状态，包装实例就会变成 fulfilled 状态；

如果所有参数实例都变成 rejected 状态，包装实例就会变成 rejected 状态。

![](https://cdn.jsdelivr.net/gh/jimdeng92/static_1@main/微信截图_20211231114616.5dmz01w3vgg0.webp)

## 参考资料

- [https://es6.ruanyifeng.com/#docs/promise](https://es6.ruanyifeng.com/#docs/promise)
