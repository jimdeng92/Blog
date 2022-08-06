# Mustache语法原理

使用过 vue 的都知道一个基本用法，就是在模板中使用变量需要用双大括号包裹变量。

vue 官方文档也有说明：
数据绑定最常见的形式就是使用“Mustache”语法 (双大括号) 的文本插值：

``` vue
<template>
  <span>Message: {{ msg }}</span>
</template>
```

vue 借鉴的就是 Mustache 这个库，Mustache 算是今天前端模板语法的祖师爷了。那么我们来一起回顾一下前端模板语法的发展历程和 Mustache 的基本原理。

## 前端模板语法的发展

为了对比各个方法的优劣，我们假设要创建一个如下的 DOM 结构：

``` html
<ul id="list">
  <li>
    <div class="hd">小明的基本信息</div>
    <div class="bd">
      <p>姓名：小明</p>
      <p>年龄：18</p>
      <p>性别：男</p>
    </div>
  </li>
</ul>
```

当然不仅仅是一个简单的 `li` ，我们假设要遍历三遍，比如后端数据是这样的：

``` js
const arr = [
  {name: '小明', age: 18, sex: '男'},
  {name: '小红', age: 14, sex: '女'},
  {name: '小白', age: 16, sex: '男'},
]
```

1. 原生 JavaScript api 创建节点

如果使用原生 JS api 的话，操作起来就会非常的复杂，我们需要不断的调用 `createElement` 和 `appendChild` ，下面的大体的实现过程：

``` js
const fragment = document.createDocumentFragment()
const list = document.getElementById('list')

for(let i = 0; i < arr.length; i++) {
  const li = document.createElement('li')

  const hdDiv = document.createElement('div')
  hdDiv.className = 'hd'
  hdDiv.innerText = arr[i].name + '的基本信息'

  const bdDiv = document.createElement('div')
  bdDiv.className = 'bd'

  const p1 = document.createElement('p')
  p1.innerText = '姓名：' + arr[i].name
  bdDiv.appendChild(p1)

  const p2 = document.createElement('p')
  p2.innerText = '年龄：' + arr[i].age
  bdDiv.appendChild(p2)

  const p3 = document.createElement('p')
  p3.innerText = '性别：' + arr[i].sex
  bdDiv.appendChild(p3)

  li.appendChild(hdDiv)
  li.appendChild(bdDiv)

  fragment.appendChild(li)
}

list.appendChild(fragment)
```

可以看到，尽管通过遍历，创建一个简单的 DOM 结构，也需要我们考虑大量的逻辑，接下来看看第二种方法。

2. `[].join()`

数组的 `join` 方法可以把数组的各个元素拼接起来，我们可以利用这一特性插入变量，同时也有比较好的可读性：

``` js
const list = document.getElementById('list')

for(let i = 0; i < arr.length; i++) {
  list.innerHTML += [
    '<li>',
    '  <div class="hd">' + arr[i].name + '的基本信息</div>',
    '  <div class="bd">',
    '    <p>姓名：' + arr[i].name + '</p>',
    '    <p>年龄：' + arr[i].age + '</p>',
    '    <p>性别：' + arr[i].sex + '</p>',
    '  </div>',
    '</li>'
  ].join('')
}
```

这样的实现方法显然要高明得多，不仅代码更简洁，同时也更易读，第三种方式与 `join` 方法类似，只不过是 ES6 的实现方式。

3. ES6 模板字符串

直接上代码：

``` js
const list = document.getElementById('list')

for(let i = 0; i < arr.length; i++) {
  list.innerHTML += `
    <li>
      <div class="hd">${arr[i].name}的基本信息</div>
      <div class="bd">
        <p>姓名：${arr[i].name}</p>
        <p>年龄：${arr[i].age}</p>
        <p>性别：${arr[i].sex}</p>
      </div>
    </li>
  `
}
```

正常来说，我们在原生 JS 环境中可以使用这种方式实现模板字符串，而不需要借助 artTemplate 等库或者使用 JS api ，毕竟这种方式更优雅而且 ES6 的模板字符串也是现在绝大部分的现代浏览器所支持的。

4. Mustache

我们再来看看 Mustache 的写法。当然，Mustache 不见得更简洁，但是它的功能更强大，而且我的这篇文章也是为了探寻 Mustache 的原理，以上不过是为了引出 Mustache 而已。

``` js
const templateStr = `
  <ul>
    {{#arr}}
      <li>
        <div class="hd">{{name}}的基本信息</div>
        <div class="bd">
          <p>姓名：{{name}}</p>
          <p>年龄：{{age}}</p>
          <p>性别：{{sex}}</p>
        </div>
      </li>
    {{/arr}}
  </ul>
`

const domStr = Mustache.render(templateStr, data)

document.querySelector('#app').innerHTML = domStr
```

可以发现，Mustache 的变量都通过双大括号包裹，这跟 Mustache 的语法很像（本来就是一样），通过在模板中直接遍历。不过需要注意的是他的数据结构有一些些变化：

``` js
const data = {
  arr: [
    {name: '小明', age: 18, sex: '男'},
    {name: '小红', age: 14, sex: '女'},
    {name: '小白', age: 16, sex: '男'},
  ]
}
```

下面，我们来实现一个简单的 Mustache 。

## Mustache 原理

开始之前，我先简单说一下 Mustache 的原理和大体步骤，好让大家有一个大概的结构：

它的核心原理是将模板字符串转化为 tokens（`parseTemplateToTokens` 方法），再将 tokens 和数据结合生成最终的 DOM 字符串（`renderTemplate` 方法）。

![mustache](https://cdn.jsdelivr.net/gh/jimdeng92/static_1/mustache.png)

主要步骤：

- 通过 `Scanner`  类将模板字符串转成最简单的二维数组；
- 通过 `nestTokens` 将模板字符串中的遍历模板利用“栈”思想生成更多维的数组，使 tokens 与模板字符串匹配层级关系；
- 通过 `renderTemplate` 将 tokens 递归处理拼接成 DOM 字符串。

注意：这个版本不处理带布尔值的判断等逻辑。

详细的代码在我的 [github 仓库](https://github.com/jimdeng92/learn-vue/tree/master/learn-mustache)。

这一次我们把数据定复杂一点，最终我们要实现的效果如下：

``` js
const templateStr = `
  <ul>
    {{#arr}}
      <li>
        <div class="hd">{{name}}的基本信息</div>
        <div class="bd">
          <p>姓名：{{name}}</p>
          <p>年龄：{{age}}</p>
          <p>性别：{{sex}}</p>
          <p>
            爱好：
            {{#hobbies}}
              <span>{{.}}</span>
            {{/hobbies}}
          </p>
        </div>
      </li>
      {{/arr}}
  </ul>
`
const data = {
  arr: [
    {name: '小明', age: 18, sex: '男', hobbies: ['游泳', '健身']},
    {name: '小红', age: 14, sex: '女', hobbies: ['旅游', '美食']},
    {name: '小白', age: 16, sex: '男', hobbies: ['读书', '打篮球']},
  ]
}

const tempStr = templateEngine.render(templateStr, data)

document.querySelector('#container').innerHTML = tempStr
```

上面已经讲过，Mustache 的核心是模板转 tokens，tokens 转 DOM。那也就是说，`render` 返回的是 DOM，而它大体要经过两个步骤：

``` js
import parseTemplateToTokens from './parseTemplateToTokens'
import renderTemplate from './renderTemplate'

window.templateEngine = {
  render(templateStr, data) {
    // 将模板转为 tokens
    const tokens = parseTemplateToTokens(templateStr)
    // 将 tokens 与数据结合生成 DOM
    const resultStr = renderTemplate(tokens, data)

    return resultStr
  }
}
```

那么，我们到底要将模板字符串转化成怎么样的 tokens 呢，来看看它的结构吧。

![tokens](https://cdn.jsdelivr.net/gh/jimdeng92/static_1/tokens.png)

每次我们都在双大括号处分割字符，是字符就是 text 类型，是变量就是 name 类型，特殊字符（如：#）就直接将特殊字符作为类型标识。可以看到，每个数组的第一项就是标识符，第二项就是模板字符串中的字段，特殊的如 # 会有第三项，它的第三项又是一个数组...

接下来，我们看看如何实现这样的 tokens，它被放在 `parseTemplateToTokens` 方法里。

``` js
import Scanner from './Scanner'
import nestTokens from './nestTokens'

export default (templateStr) => {

  const scanner = new Scanner(templateStr)

  const tokens = []

  while(!scanner.eos()) { // 不是最后的字符
    var words = scanner.scanUtil('{{') // 返回 {{ 前的字符
    if (words !== '') {
      tokens.push(['text', words])
    }
    scanner.scan('{{') // 跳过 {{

    words = scanner.scanUtil('}}')
    if (words !== '') {
      if (words.charAt(0) === '#') {
        tokens.push(['#', words.substring(1)])
      } else if (words.charAt(0) === '/') {
        tokens.push(['/', words.substring(1)])
      } else {
        tokens.push(['name', words])
      }
    }
    scanner.scan('}}')
  }

  return nestTokens(tokens)
}
```

我们使用了 `while` 循环，一个个字符的查找，直到找完所有的字符，当遇到左大括号时将之前找到的字符 `push` 到一个空数组中（至于怎么返回左大括号前的字符可以上我的 github 仓库，我不想文章的代码过长，容易看晕 : ）。然后跳过左大括号，继续向后查找，到找到右大括号时再将 `['name', words]`  push 到 tokens 里，然后跳过右大括号，如此循环...。

这个时候我们得到的就是一个二维数组，并不像我们先前所说的多维数组：

![tokens_2](https://cdn.jsdelivr.net/gh/jimdeng92/static_1/tokens_2.png)

要将这样的一个二维数组变成多维数组需要一些编程技巧，那就是 **栈**。

我们需要遍历 tokens，在遇到 token[0] 是 # 是入栈，在遇到 token[0] 是 / 时出栈，具体实现如下：

``` js
export default (tokens) => {
  const nestedTokens = [] // 结果数组

  const sections = [] // 栈结构
  let controller = nestedTokens // 控制器，初始值为结果数组

  for(let i = 0; i < tokens.length; i++) {
    const token = tokens[i]

    switch(token[0]) {
      case '#':
        sections.push(token) // 入栈
        controller.push(token) //
        controller = token[2] = [] // 改变控制器位置
        break;
      case '/':
        sections.pop() // 出栈
        controller = sections.length ? sections[sections.length - 1][2] : nestedTokens // 改变控制器位置
        break;
      default:
        controller.push(token) // 推入控制器
    }
  }

  return nestedTokens
}
```

除了**栈**的思想以外，这里的 controller 也十分重要。

完成这一步，我们终于得到我们需要的 tokens 了，接下来完成第二步 --- 将 tokens 和 data 结合起来，生成最终的 DOM。

``` js
// renderTemplate.js
import lookup from './lookup'

export default function renderTemplate(tokens, data) {
  let resultStr = ''

  for(let i = 0; i < tokens.length; i++) {
    let token = tokens[i]
    if (token[0] === 'text') {
      resultStr += token[1]
    } else if (token[0] === 'name') {
      if (token[1] === '.') { // 处理 {{.}}
        resultStr += data
      } else {
        resultStr += lookup(data, token[1]) // 处理 {{a.b.c}}
      }
    } else if (token[0] === '#') {
      const arr = lookup(data, token[1]) // 内部数组
      for(let j = 0; j < arr.length; j++) {
        resultStr += renderTemplate(token[2], arr[j])
      }
    }
  }

  return resultStr
}
```

这就大功告成了，其实核心代码并不多，主要是理解 mustache 的创新思路，以及它的处理流程，一步步来解决问题。自然而然就能推出整个过程。
