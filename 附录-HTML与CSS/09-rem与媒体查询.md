# 02-rem 与媒体查询

## 一 媒体查询

媒体查询（Media Query）是 CSS3 引入的新技术，可以针对不同的屏幕尺寸设置不同的样式，在重置浏览器大小的过程中，页面也会根据浏览器的宽度和高度重新渲染页面！

示例：

```css
/*
声明媒体查询：@media
参数一：mediatype，媒体类型，值有：
        all：用于所有设备
        print：用于打印机和打印预览
        screen：用于电脑、平板、手机等

参数二： 用于连接参数1与参数二，为 and not only 等值
        and：可以将多个媒体特性连接到一起，即： 且
        not：排除某个媒体类型，即：非，可以省略
        only：指定某个特性的特性，可以省略
参数三： media feature 特性，必须有小括号包含
        width: 可视区宽度
        min-width：可视区最小宽度
        max-width：可视区最大宽度
*/
@media screen and (max-width: 800px) {
  /* 在屏幕中，且设置最大宽度为800px */
  body {
    background-color: pink;
  }
}

@media screen and (max-width: 500px) {
  body {
    background-color: green;
  }
}
```

## 二 rem 适配方案

### 2.0 rem 适配布局概念

px 是一个实际的像素大小，rem（root em）、em 都是相对大小：

- em：相对于当前元素的字体大小，没有则相对于父级。比如父元素的字体为 5px，子元素为 2em，则子元素的字体为 10px。
- rem：相对于 html 根元素字体大小（默认为 16px）。比如 html 设置了 `font-size=10px`，若某个非根元素设置 `width:2rem;` 换算为 px 就是 20px。

当使用 rem 作为单位时，只要 html 元素中的字体大小发生改变，那么整体的布局就会相应发生改变，其适配的核心方案是随着屏幕的变化，字体发生相应变化，界面进行等比例缩放。

rem 可以用来解决布局中一些大小问题，如：

- 传统布局、flex 布局中，文字都不能随着屏幕大小变化而变化
- 流式布局和 flex 布局主要针对宽度进行布局，高度不能很好的定义
- 在屏幕发生变化时，元素的宽高不能很好的进行等比例缩放

这里就涉及 html 标签的 font-size 大小动态设置问题：

```txt
通常会将屏幕划分为15等份，根据需求也可以设置为10等份，20等份，用页面元素大小除以不同的html字体大小，会发现其比例相同：

在设计稿为750px时，其html字体的大小为：750/15=50px
在设备中为320px时，其html字体的大小为：320/15=21.33px

现在假设有一个100*100px的页面元素：
在750屏幕下，html字体大小为50.00px，转换为rem：100/50.00，即：2rem*2rem，宽高比例是1比1
在320屏幕下，rem值上面已经写好是2*2，但是其字体是：21.33px，实际像素2rem即：42.66px * 42.66px，宽高比例没变！
```

由上得出：**`rem值` = `页面元素px` / `html的font-size`**。

贴士：如果不设置字体大小，1rem=16px

### 2.1 rem 实现方式一：js 控制

现在的核心问题就是根据屏幕自动计算 rem 的值，下面的脚本可以实现自动改变字体大小。有了该脚本，只需要将设计稿（750px）量出的像素值除以 100 即可得到 rem 的值.这里要注意，html 根元素的字体大小改变会引起一个 bug：图片与文件间距会出现变化，可以为 body 设置固定大小即可：

```html
<style>
  * {
    margin: 0;
    padding: 0;
  }
  body {
    font-family: Arial, Helvetica, sans-serif;
    font-size: 16px;
  }

  .header {
    background: #ff5555;
    height: 0.82rem;
    width: 100%;
  }
</style>

<body>
  <div class="header"></div>

  <script>
    ;(function (doc, win) {
      let docElement = doc.documentElement
      let resizeEvent = 'orientationchange' in window ? 'orientationchange' : 'resize'

      let recalc = function () {
        let clientWidth = docElement.clientWidth
        if (!clientWidth) return

        // 设计稿基准为750px
        if (clientWidth >= 750) {
          docElement.style.fontSize = '100px'
        } else {
          docElement.style.fontSize = 100 * (clientWidth / 750) + 'px'
        }
      }

      if (!doc.addEventListener) return
      win.addEventListener(resizeEvent, recalc, false)
      doc.addEventListener('DOMContentLoaded', recalc, false)
    })(document, window)
  </script>
</body>
```

### 2.2 rem 适配方式三：配合媒体查询

常见需要配置的：

```css
html {
  font-size: 50px;
}

@media screen and (min-width: 320px) {
  html {
    font-size: 21.3333px;
  }
}

@media screen and (min-width: 360px) {
  html {
    font-size: 24px;
  }
}

/* iphone678 */
@media screen and (min-width: 375px) {
  html {
    font-size: 25px;
  }
}

/* 其他需要设置的常见屏幕：384 400 414 424 480 540 720 750 */
```

当样式繁多时，使用媒体查询引入资源可以更好的实现移动端布局：

```html
/* 针对不同的屏幕尺寸，引入不同的css资源 */
<link rel="stylesheet" href="./small.css" media="screen and (min-width:320px)" />
<link rel="stylesheet" href="./big.css" media="screen and (min-width:640px)" />
```

### 2.3 rem 实现方式二：vw 布局

vw 布局可以看做的是 rem 的进化版，比上述 js 控制的方式更加简单，无需对字体大小进行控制，但是只兼容 iOS8、Android4.4 以上系统。

vw 布局是将屏幕划分为 100 份，即屏幕是 100vw（也即 vw 是 1%的屏幕宽度），换算到在 750px 设计稿中就是 `750px=100vw`，1px 就是 `0.1333333333vw`。为了方便计算，实际开发中根元素不可能是 1px，放大 100 倍：

```html
<style>
  html {
    font-size: 13.33333333vw;
  }
  @media (min-width: 750px) {
    html {
      font-size: 100px;
    }
  }
</style>
```

由于 vw 布局是自己将屏幕等份划分，所以也就不再依赖与 JS 脚本、媒体查询来控制字体大小，开发更方便。

## 三 企业级 rem 适配总结

目前有两种常见的实践方案：

- less+媒体查询+rem：一般采用标准尺寸 750px，页面元素的 rem 值=750 像素下的 px 值/html 文字大小
- flexible.js+rem：更简便，该库由淘宝推出，有了其支持，不再需要对不同屏幕进行媒体查询。其原理是将当前设备划分为了 10 等份，在不同设备下比例一致。如当前设计稿是 750px，那么只需要把 html 的字体设置为 750px/10 即可，当前元素的 rem 值就是：页面元素的 px 值/75，其他的交给了库自己运算

贴士：vscode 插件 cssrem 可以快速帮助运算。

## 四 CSS 预处理语言

### 4.1 CSS 预处理语言概念

CSS 由于其不支持变量、函数运算等缺陷，在开发时候很不便，目前市面上有三种 CSS 扩展语言（预处理器）：

- Less
- Sass
- Stylus

扩展语言并没有减少 CSS 的功能，而是在现有语法基础上，添加一些特性，如：

- 变量
- Mixin
- 运算
- 函数

Less 中文网址：<http://lesscss.cn>

### 4.2 Less 简单使用

HTML 页面无法直接使用 Less，需要将 Less 转换为 CSS，VSCode 中安装 `Easy LESS` 插件即可在编写 Less 时自动将 less 转换为 css。

新建后缀为 less 的文件：

```less
// index.less less可以使用 // 注释

// @ 定义一个变量
@baseColor: yellowgreen;
// 使用变量
body {
  background-color: @baseColor;
}

// less 嵌套
#app {
  font-size: 14px;
  .header {
    height: 200px;
    .header-left {
      bacground-color: @baseColor;
    }
    .header-right {
      bacground-color: #fff;
    }

    a {
      color: red;
      // &符号被解析为伪类、伪元素、交集选择器
      &:hover {
        color: #fff;
      }
    }
  }
}
```

Less 支持运算： `+ - * /`，运算符中间左右需要使用空格隔开。如果两个不同单位之间进行运算，运算结果的单位为第一个值的单位。如果两个值之间只有一个有单位，则运算结果取该单位。

```less
@border:5px + 5;

body {
  width: 200px - 50;
}
```

less 引入另外一个 css：

```css
/* 引入common.less */
@import 'common';
```

less 中运算 rem：

```less
@baseFont: 50;
.box {
  height: 100rem / @baseFont; /*高度为100px*/
}
```
