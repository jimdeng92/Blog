---
title: 自定义全局MessageBox插件
date: 2021-09-15 18:20:35
permalink: /pages/81c604/
categories: 
  - 开发者手册
  - Vue
tags: 
  - 
---

这两天用 Element 搞开发的过程中，由于对系统有一定的 UI 要求，它的弹窗是长这样的：

![](https://cdn.jsdelivr.net/gh/jimdeng92/static_1/微信截图_20210531105303.png)

我是翻遍了饿了么的文档都没能找到满足我需求的 API，而且弹窗有复用要求。

**关键点在于我不能简单的把它封装成组件，毕竟调用组件的事件和组件的 confirm 事件是不相同的，最好是有一个像饿了么 MessageBox 一样的机制，通过 JS 调用，弹窗内容可以自定义，返回的需要是 `Promise`。**

Vue 的插件机制有非常强大的功能，它可以：

- 添加全局方法或者 property。如：[vue-custom-element](https://github.com/karol-f/vue-custom-element)
- 添加全局资源：指令/过滤器/过渡等。如 [vue-touch](https://github.com/vuejs/vue-touch)
- 通过全局混入来添加一些组件选项。如 [vue-router](https://github.com/vuejs/vue-router)
- 添加 Vue 实例方法，通过把它们添加到 `Vue.prototype` 上实现。
- 一个库，提供自己的 API，同时提供上面提到的一个或多个功能。如 [vue-router](https://github.com/vuejs/vue-router)

首先，我像定义普通组件一样定义好弹窗内容，这里为了节省时间（懒），我使用了饿了么的弹窗组件：

::: details 点击查看代码
``` vue
<!--
  @description: 自定义 MessageBox 组件
  @params
    title: 弹窗标题（默认：提示）
    type: 弹窗类型（默认：primary）可选：primary，success，warning，danger
    content: 内容
    confirmButtonText: resovle 按钮（默认：确定）
    cancelButtonText: reject 按钮（默认：取消）
    iconText: 图标文本内容（默认： ！）
  @use:
    this.$showMessage({
      title: '确定删除？'
    })
  @remark: 无必填项
-->

<template>
  <el-dialog
    :visible.sync="visible"
    width="400px"
    top="35vh"
    append-to-body
    :show-close="false">
  <div class="dialog-body">
    <div :class="[ 'dialog-body-icon', type ]"> {{iconText}} </div>
    <div class="dialog-body-main">
      <p class="main-title">{{title}}</p>
      <div class="main-content">
        {{content}}
      </div>
    </div>
  </div>
  <span slot="footer">
    <el-button :type="buttonType" @click="handleEnsure">{{confirmButtonText}}</el-button>
    <el-button @click="handleClose">{{cancelButtonText}}</el-button>
  </span>
</el-dialog>
</template>

<script>

export default {
  name: 'MessageBox',
  props: {
    iconText: {
      type: String,
      default: '!'
    },
    title: {
      type: String,
      default: '提示'
    },
    type: {
      type: String,
      default: 'primary'
    },
    content: {
      type: String,
      default: ''
    },
    confirmButtonText: {
      type: String,
      default: '确定'
    },
    cancelButtonText: {
      type: String,
      default: '取消'
    }
  },
  data () {
    return {
      visible: false,
      reject: null,
      resolve: null
    }
  },
  computed: {
    buttonType () {
      let t = 'primary'
      switch (this.type) {
        case 'primary':
          t = 'primary'
          break
        case 'success':
          t = 'success'
          break
        case 'warning':
          t = 'warning'
          break
        case 'danger':
          t = 'danger'
          break
        default:
      }
      return t
    }
  },
  methods: {
    handleShow () {
      this.visible = true
      return new Promise((resolve, reject) => {
        // 打开弹窗时创建 primise 并且保持状态，点击 确定或取消 再扭转状态
        this.resolve = resolve
        this.reject = reject
      })
    },
    handleClose () {
      this.visible = false
      this.reject('cancel')
      document.body.removeChild(this.$el)
    },
    handleEnsure () {
      this.visible = false
      this.resolve('confirm')
    }
  }
}
</script>

<style lang="less" scoped>
// css 省略
</style>
```
:::

上述代码唯一的特别之处只有在 `handleShow` 方法中返回了一个 `Promise` 的实例，因为我们需要在通过这个方法调起组件，同时保持 `Promise` 的状态，只有在点击确定或者取消时才去执行 `resolve/reject` 方法。

看看接下来我们该怎么注册插件，创建一个 JS 文件：

::: details 点击查看代码
``` js
import msgboxVue from './MessageBox'

const MessageBox = {} // 定义插件对象

// Vue.js 的插件应该暴露一个 install 方法。这个方法的第一个参数是 Vue 构造器，第二个参数是一个可选的选项对象
MessageBox.install = function (Vue) {
  // 使用基础 Vue 构造器，创建一个“子类”。参数是一个包含组件选项的对象
  // 需要关注两个点：
  // 第一，.vue 文件返回的就是一个组件对象；
  // 第二，extend 方法返回的是一个 Vue 构造器的“子类”，也就是说，MessageBoxInstance 也是一个构造器
  const MessageBoxInstance = Vue.extend(msgboxVue)

  let instance

  const initInstance = () => {
    instance = new MessageBoxInstance() // 实例化 vue 实例
    let msgBoxEl = instance.$mount().$el // 渲染为文档之外的的元素
    document.body.appendChild(msgBoxEl)
  }

  // 在Vue的原型上添加实例方法，以全局调用
  Vue.prototype.$showMessage = (options) => {
    if (!instance) {
      initInstance()
    }

    if (typeof options === 'string') {
      instance.content = options
    } else if (typeof options === 'object') {
      Object.assign(instance, options)
    }

    return instance.handleShow()
      .then(val => {
        instance = null
        return Promise.resolve(val)
      })
      .catch(err => {
        instance = null
        return Promise.reject(err)
      })
  }
}

export default MessageBox
```
:::

如此，就实现了自定义的 MessageBox 。此时它是作为一个插件存在，在 main.js 中使用插件就可以了。

``` js
import MessageBox from './components/messageBox/index.js'

Vue.use(MessageBox)
```

使用的时候直接调用 `this.$showMessage({...})` 就可以了，点击确认就会进入 `then`，点击取消就会进入 `catch` 并抛出错误 'cancel'。
