## 闭包（closure）

> 所谓闭包指的是有权访问另一个函数作用域中变量的函数。创建闭包的常见方式，就是在一个函数内部创建另一个函数。本质上，闭包就是将函数内部和函数外部连接起来的一座桥梁。

``` js
function fn1() {
  var n = 999
  return function() {
    alert(n)
  }
}
var result = fn1()
result()
```

上面的例子fn1函数中返回了一个匿名函数，这个匿名函数访问了fn1中的变量n。即使这个匿名函数被返回，然后在其他地方调用了，但它仍然可以访问到n。因此，我们可以在函数体外访问到函数体内的变量。这就是闭包的两大用处之一——读取函数内部的变量，另一个是让这些变量始终保存在内存中。看一下这个例子就明白了：

```js
function fn1() {
  var n = 999
  // 注意nAdd是全局变量
  nAdd = function() {
    n += 1
  }
  function fn2() {
    console.log(n)
  }
  return fn2
}
var result = fn1()
result() // 999
nAdd()
result() // 1000
```

上例中fn2被赋值给了一个全局变量，因此它始终在内存中，而fn2的存在依赖于fn1，因此fn1也始终在内存中。

下面的例子会返回一个函数数组，我们期望函数返回当前位置的索引：

```js
function createFunctions() {
  var result = []
  for(var i = 0; i < 10; i++) {
    result[i] = function() {
      return i
    }
  }
  return result
}
createFunctions()[0]() // 10
```

实际上，无论是第几项返回的都是10。因为每个函数的作用域链中都保存着createFunctions()函数的活动对象，所有他们引用的都是同一个变量i。要使函数符合预期，可以这样：

```js {4,5,6,7,8}
function createFunctions() {
  var result = []
  for(var i = 0; i < 10; i++) {
    result[i] = function(num) {
      return function() {
        return num
      }
    }(num)
  }
}
```

这一次，我们没有直接将闭包赋值给数组。而是定义了一个匿名函数，并将立即执行函数的结果赋值给了数组。由于函数参数是按值传递的，所以就会将变量i的当前值复制给参数num。这样一来，result数组中的每个函数都有自己num变量的一个副本，因此就可以返回各自不同的值了。

严格来说，内存泄漏并不是闭包的缺点，这是IE9之前的版本对JScript和COM对象使用不同的垃圾回收例程所导致的。由于闭包的特性，函数体内的变量会被保存在内存中不被释放，因此频繁的使用闭包还是会造成大量的内存消耗，影响网页性能。解决办法是在退出函数前，将不使用的局部变量删除。

## 原型和原型链

[原型和原型链](https://imlinhe.com/pages/98254c/)

## this

> 关于this是一个老生常谈的话题了，记住一点：**this永远指向最后调用它的那个对象。**
先来一个简单的例子：

```js
var name = 'windowName'
function fn() {
  var name = 'linhe'
  console.log(this.name)
}
fn() // windowName
```

用**this永远指向最后调用它的那个对象**来解释一下，前面没有调用对象那么就是全局对象window，因此在全局找name然后就找到了windowName。

再换一个例子看看：

```js
var name = 'windowName'
var obj = {
  name: 'linhe',
  fn: function() {
    console.log(this.name)
  }
}
console.log(obj.fn()) // linhe
```

还是用这句话就很好理解了，调用fn的对象是obj，所以this.name就是linhe。

改动一下看看：

```js
var name = 'windowName'
var obj = {
  fn: function() {
    console.log(this.name)
  }
}
var fn = obj.fn
fn() // windowName
```

看到这里可能会有点疑惑，fn不是指向obj.fn吗？再仔细看看**this永远指向最后调用它的那个对象**，在将obj.fn重新赋值的时候其实并没有调用对象的这个方法，真正调用是在最后一行代码，还是全局调用，所以是windowName。

到这里，不管什么样的句式应该都不成问题了。

```js
var name = 'windowName'
function fn() {
  var name = 'linhe'
  innerFunc()
  function innerFunc() {
    console.log(this.name) // windowName
  }
}
```

看看吧，就算在函数体内调用，没有对象调用函数那就是全局调用。

#### 改变this指向的方法：

- 使用ES6箭头函数；
- 在函数内部保存this(_this = this);
- 使用call、apply、bind；
- new实例化一个对象。

```js
var name = 'windowName'
var obj = {
  name: 'linhe',
  func1: function() {
    console.log(this.name)
  },
  func2: function() {
    setTimeout(function() {
      this.func1()
    }, 1000)
  }
}
obj.func2() // this.func1 is not a function
```

setTimeout是全局方法，它内部的回调函数指向全局，而全局没有func1方法，因此就报错了。

按照上面说的几点，改变this的指向试试：

箭头函数：~~**箭头函数的this始终指向函数定义时的this，而非执行时。**~~**箭头函数内部没有 this，它始终沿用上一个作用域栈中的this。**

```js
var name = 'windowName'
var obj = {
  name: 'linhe',
  func1: function() {
    console.log(this.name)
  },
  func2: function() {
    setTimeout(() => {
      this.func1()
    }, 1000)
  }
}
obj.func2() // linhe
```

在函数内部使用变量存储this的指向

```js
var name = 'windowName'
var obj = {
  name: 'linhe',
  func1: function() {
    console.log(this.name)
  },
  func2: function() {
    var _this = this
    setTimeout(function() {
      _this.func1()
    }, 1000)
  }
}
obj.func2() // linhe
```

使用call、apply、bind

```js
// call
var name = 'windowName'
var obj = {
  name: 'linhe',
  func1: function() {
    console.log(this.name)
  },
  func2: function() {
    setTimeout(function() {
      this.func1()
    }.call(obj), 1000)
  }
}
obj.func2() // linhe
// apply
var name = 'windowName'
var obj = {
  name: 'linhe',
  func1: function() {
    console.log(this.name)
  },
  func2: function() {
    setTimeout(function() {
      this.func1()
    }.apply(obj), 1000)
  }
}
obj.func2() // linhe
// bind
var name = 'windowName'
var obj = {
  name: 'linhe',
  func1: function() {
    console.log(this.name)
  },
  func2: function() {
    setTimeout(function() {
      this.func1()
    }.bind(obj), 1000)
  }
}
obj.func2() // linhe
```

说一下上述三个方法的第一个参数，它是一个对象。call、apply、bind都是函数的方法，调用方法改变函数体内this的指向，改为指向第一个参数（对象）,在非严格模式下，第一个参数为null或undefined时，函数体内的this则指向全局对象。

**这里有一点需要注意：上面的call、apply返回的是函数的调用，也就是执行`this.func1()`，而`this.func1()`的返回值是undefined，这样的代码在真正的业务中并不适用，因为这会导致setTimeout立即执行。而bind返回的是setTimeout回调函数的拷贝，在业务中逻辑合理，会延迟一秒执行func1，打印出linhe。**

## 手写call、apply、bind

:top: 上面说到关于this指向问题时已经谈到过这三个方法可以改变函数的this指向，那么先来看看call、apply、bind各自的定义和区别先：

#### call

call()方法使用指定的this值和单独给出的一个或多个参数来调用一个函数。

语法：

> *function*.call(thisArg, arg1, arg2, ...)
参数thisArg是可选的，它指的是function函数运行时使用的this值。在非严格模式下，当thisArg为null或undefined时，this指向全局对象，在浏览器中就是window。

参数`arg1, arg2, ...`是要传入function的参数列表。

#### apply

apply和call方法类似，只有一个区别，就是call()方法接受的是一个参数列表，而apply()方法接受的是一个包含多个参数的数组。

语法：

> *function*.apply(thisArg, [arg1, arg2, ...])
#### bind

bind()方法创建一个新函数，在bind()被调用时，这个新函数的this被指定为bind()的第一个参数，而其余参数将作为新函数的参数，供调用时使用。bind()返回原函数的拷贝，并拥有指定的this值和初始参数。

语法：

> *function*.bind(thisArg[, arg1 [, arg2[, ...]]])
了解了概念之后，我们再来看看手写call、apply、bind的实现方式。

#### 手写call

先来一个原生call的使用方式：

```js
let Person = {
  name: 'linhe',
  say: function() {
    console.log(this.name)
  }
}
Person.say() // linhe
let p1 = {
  name: 'Tom'
}
// 原生call
Person.say.call(p1) // Tom
```

通过上面的示例我们发现：如果p1有say()方法，执行p1.say()可以得到和Person.say.call(p1)同样的结果。没错这就是call的实现原理。

```js
Function.prototype.myCall = function(context) {
  // context就是上例的p1对象
  // this结合上例就是Person.say()方法
  context.say = this
  // 立即执行
  context.say()
}
Person.say.myCall(p1) // Tom
```

就是这么简单，不过还有几点需要完善：

- 支持多个参数；
- 给上下文定义的函数要保持唯一；
- 调用完函数后要进行删除。

```js
Function.prototype.myCall = function() {
  // 没有参数或者为null指向window
  let context = arguments[0] == null ? window : arguments[0]
  // 创建一个唯一的方法名，防止被调用的对象本来有同名方法而被覆盖
  let fn = Symbol('fn')
  // 把要借用的方法拷贝到对象方法上
  context[fn] = this
  // 截取参数
  let arg = [...arguments].slice(1)
  // 立即调用
  const result = context[fn](...arg)
  // 删除
  delete context[fn]
  // 返回结果
  return result
}
// 测试
let person = {
  name: 'linhe',
  say(age) {
    console.log(`我的名字：${this.name}，我的年龄：${age}岁`)
  }
}
let p1 = {
  name: 'Tom'
}
person.say.myCall(p1, 18)
person.say.myCall(null, 18)
```

#### 手写apply

```js
Function.prototype.myApply = function() {
  let context = arguments[0] == null ? window : arguments[0]
  let fn = Symbol('fn')
  context[fn] = this
  // apply第二个参数是数组
  let args = arguments[1] || []
  const result = context[fn](...args)
  delete context[fn]
  return result
}
```

#### 手写bind

```js
Function.prototype.myBind = function() {
  let context = arguments[0] == null ? window : arguments[0]
  //返回一个绑定this的函数，我们需要在此保存this
  let self = this
  // 可以支持柯里化传参（fn.bind(a)(b)），保存参数
  let args = [...arguments].slice(1)
  // 返回函数
  return function() {
    let newArgs =
    return self.apply(context, [...args, ...arguments])
  }
}
```

## 宏任务和微任务
