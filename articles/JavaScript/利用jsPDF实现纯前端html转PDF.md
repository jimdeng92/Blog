---
title: 利用jsPDF实现纯前端html转PDF
date: 2021-11-04 14:44:53
permalink: /pages/9c412d/
categories: 
  - 开发者手册
  - JavaScript
tags: 
  - 
---

[jsPDF](https://github.com/parallax/jsPDF) 是前端一个非常优秀的库，它可以自定义 PDF 内容，包括在 PDF 中添加文字，图片，图形，表格等。

它还能将页面中的 DOM 直接转为 PDF 中的内容。这里我想要分享一下包括怎样初始化库并且使用，以及在使用 jsPDF 中遇到的问题做个整理，供需要的童鞋参考。首先说明一下，我的使用环境是 @vue/cli，其他环境也适用，基本类似。

## 安装和使用 jsPDF

``` shell
npm install jspdf --save
# or
yarn add jspdf
```

安装完成后，在需要的组件中使用：

``` vue
<script>
import { jsPDF } from 'jspdf'

export default {
  // ...
  methods: {
    downloadPDF() {
      // 更多参数参考文档：http://raw.githack.com/MrRio/jsPDF/master/docs/jsPDF.html
      var doc = new jsPDF({
        orientation: 'l', // pdf页方向： p 竖向 l 横向
        unit: 'px', // 单位
        format: [1600, 900.01], // 默认 a4，可自定义尺寸，竖向多0.01是防止多一个空白页
        encryption: { // 加密
          ownerPassword: 'owner', // 管理员密码（所有权限）
          userPassword: 'user' // 用户密码
          // userPermissions: ['print', 'modify', 'copy', 'annot-forms'] // 用户权限列表
        }
      })

      // html 转 pdf
      doc.html(this.$refs.pdfContent, {
        callback: (doc) => {
          doc.save()
        }
      })
    }
  }
}
</script>
```

有些博主会在 DOM 超出一屏时使用 `addPage()`，但我使用时 jsPDF 实际上已经自动分页了，我们只需要在 $refs.pdfContent 中定义多个 1600 * 900 的子元素就能完美适配，实现起来非常简单。

## 一些坑

### 1、PDF 生成后乱码

> 原因：jsPDF 默认不支持中文。

解决办法肯定有，那就是自己引用编译后的字体包。步骤：

下载免费商用字体-[阿里巴巴普惠体](https://ics.alibaba.com/project/Hn8mXx)。

下载 jsPDF 最新版本源码，打开项目下的 fontconverter/fontconverter.html 文件，**上传字体的 ttf 版本。（注意只能用ttf版本的字体）**

::: center
![fontconverter.html截图](https://cdn.jsdelivr.net/gh/jimdeng92/static_1/微信截图_20211104153313.png)
:::

点击 Create 后会生成如下格式的js文件：

``` js
import { jsPDF } from "jspdf"

var font = 'AAEA...'

var callAddFont = function () {
this.addFileToVFS('Alibaba-PuHuiTi-Regular-normal.ttf', font);
this.addFont('Alibaba-PuHuiTi-Regular-normal.ttf', 'Alibaba-PuHuiTi-Regular', 'normal');
};
jsPDF.API.events.push(['addFonts', callAddFont])
```

可是这文件足足有12M，我们需要把它与项目分离出来，修改一下，只保留变量，这样我就可以把文件放 CDN 或者 /static 目录下了。

``` js
var AlibabaPuHuiTiRegular = 'AAEA...'
```

在 index.html 中引入 js，此时变量就挂载在了全局。我们来使用字体：

``` js
jsPDF.API.events.push(['addFonts', function () {
  const {AlibabaPuHuiTiRegular} = window
  this.addFileToVFS('AlibabaPuHuiTiRegular.ttf', AlibabaPuHuiTiRegular)
  this.addFont('AlibabaPuHuiTiRegular.ttf', 'AlibabaPuHuiTiRegular', 'normal')
}])

var doc = new jsPDF({
  // ...
})

doc.setFont('AlibabaPuHuiTiRegular', 'normal')

doc.html(this.$refs.pdfPageContent, {
  callback: (doc) => {
    doc.save()
  }
})
```

设置完成后还需要在css中使用：定义 `$refs.pdfPageContent` 的 `font-family: 'AlibabaPuHuiTiRegular'`。这样在下载 pdf 乱码的问题就解决了。

PS:  

- 设置 `font-family` 并不能改变 web 页面中的字体，只是供 jsPDF 在生成时使用。
- 如果包含粗体/斜体，需要生成新的字体并引入，通过修改 jsPDF 的 'normal' 字段并没有什么用，也无法通过 css 的 `font-weight` 修改。

### 二、背景图片无法显示

解决办法：不使用背景图片，使用 `<img />` 定位解决。

### 三、canvas 背景是黑色

解决办法：在画 canvas 时首先画一个跟背景一样的底色。

### 四、伪元素定位问题

在元素设置了行高后，伪元素定位不正确。

解决办法：不使用行高，用 padding。

## 特别的话

还有一些其他的适配问题没有解决的，比如：部分字会重叠、有些表现在 win 和在 Mac 上不一致、opacity 不生效、transform 后盒子内的文字消失等等。

特别需要说明的是，我利用了 opacity 不生效的 BUG（也不能算 BUG，也许是限制吧），在用户点击下载时弹出一个透明的遮罩，渲染完成后再真正去下载。这里的好处主要还是方便迭代，只需要注释一个 css 属性就能看到真实 DOM，并且不影响生成 PDF。
