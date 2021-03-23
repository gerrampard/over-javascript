# 04-Ajax

## 一 表单方式发送请求

在 Ajax 诞生之前，如果网速很慢，页面加载时间很长，就会导致用户一直在等待，而无法进行别的操作。在带表单的网页中，表单提交后，如果出现内容不合法，则会重新渲染页面，之前填写的内容就会消失。这些都是用户体验极差的现象，虽然可以通过一些手段避免，但是实现起来较为复杂。

## 二 Ajax 概述

Ajax 即 `Asynchronous Javascript And XML`（异步 JavaScript 和 XML），可以实现页面无刷新更新数据，极大提高了用户体验！

异步与同步：

- 异步：某段程序执行时不会阻塞其它程序执行，其表现形式为程序的执行顺序不依赖程序本身的书写顺序，相反则为同步
- 同步：必须等待前面的任务完成，才能执行后面的任务

Ajax 的本质仍然是 HTTP 协议上的网络通信，只不过是异步方式，其核心对象是：XMLHttpRequest（简称 XHR）。

## 三 准备服务器环境

Ajax 是客户端（浏览器）与服务端通信的方式，自然少不了服务端的参与。这里我们可以按照如下方式启动一个 Node 服务器。

首先需要下载并安装 NodeJS，进入网址<https://nodejs.org>，点击 LTS（长久支持）标识的安装包，下载后，一直下一步即可。

创建一个 NodeJS 项目，如下所示：

![node项目](../images/net/ajax-01.png)

package.json 内容如下：

```json
{
  "name": "demo-ajax",
  "version": "1.0.0",
  "description": "",
  "main": "app.js",
  "scripts": {
    "dev": "node app.js"
  },
  "dependencies": {
    "body-parser": "^1.19.0",
    "express": "^4.17.1"
  }
}
```

app.js 是 NodeJS 项目的核心文件（入口文件），代码如下：

```js
const express = require('express') // 引入web框架 express
const path = require('path') // 引入 路径处理模块 path
const bodyParser = require('body-parser') // post 请求解析模块
const formidable = require('formidable') // 解析 FormData 的模块

const app = express() // 创建web服务器

// 静态资源目录
app.use(express.static(path.join(__dirname, 'public')))

// post 请求参数设置
let urlencodedParser = bodyParser.urlencoded({ extended: false })
let jsonParser = bodyParser.json()

app.get('/hi', (req, res) => {
  res.send({ hi: 'hello get' })
})

app.post('/hi', (req, res) => {
  res.send({ hi: 'hello post' })
})

app.get('/getDemo', (req, res) => {
  console.log('GET请求参数:', req.query)
  res.send({ cid: 1001, title: '新闻一', content: '内容一一一...' })
})

app.post('/postDemo', jsonParser, (req, res) => {
  console.log('POST请求参数:', req.body)
  res.send([{ cid: 1001, title: '新闻二', content: '内容二一一...' }])
})

app.post('/formDataDemo', (req, res) => {
  const form = new formidable.IncomingForm()
  form.parse(req, (err, fileds, files) => {
    if (err) {
      res.send({ error: err })
      return
    }
    // fileds 保存了普通键值对，files保存了上传的文件
    console.log('FormData参数:', fileds)
    res.send([{ cid: 1001, title: '新闻三', content: '内容三一一...' }])
  })
})

app.post('/uploadDemo', () => {
  const form = new formidable.IncomingForm()
  form.uploadDir = path.joind(__dirname, 'uploads')
  form.parse(req, (err, fileds, files) => {
    if (err) {
      res.send({ error: err })
      return
    }
    res.send([{ cid: 1001, title: '新闻三', content: '内容三一一...' }])
  })
})

app.get('/crosDemo', (req, res) => {
  res.send("let user = {'uid':'1001'}")
})

app.get('/crosDemo2', (req, res) => {
  // 获取回调函数名称
  let callback = req.query.callback

  // 定义要返回的数据
  let data = JSON.stringify({ uid: '1001' })

  // 返回数据
  res.send(`${callback}(${data});`)
})

app.listen(3000, () => {
  console.log('服务器启动成功')
})
```

在学习 Ajax 时，只需要在 public 文件夹下的 `index.html` 内书写示例即可。

运行：

```txt
# 进入项目根目录

# 安装依赖
npm i

# 运行项目
npm run dev
```

## 四 Ajax 示例

贴士：这些示例都需要启动配置好的 NodeJS 服务器。

在 public 文件夹的 index.html 中书如下 ajax（仍然使用接口：`/hi`）：

```html
<button id="btn">点击执行Ajax</button>
<script>
  let btn = document.querySelector('#btn')
  btn.onclick = function () {
    // 1 创建 Ajax 对象。IE6 中对象为：ActiveXObject("Microsoft.XMLHTTP");
    let xhr = new XMLHttpRequest()
    // 2 设置请求方式、请求地址
    xhr.open('get', 'http://localhost:3000/hi')
    // 3 发送请求
    xhr.send()
    // 4.获取服务器端响应的数据：由于 xhr.send() 是异步的，所以后面只能用事件方式监听
    xhr.onload = function () {
      // 返回结果是字符串，需要反序列化为 JSON对象
      let data = JSON.parse(xhr.responseText)
      console.log('data: ', data)
    }
  }
</script>
```

点击按钮即可执行 Ajax。

## 五 Ajax 中的一些细节

### 5.1 GET 与 POST

在使用表单提交请求时，请求参数会被浏览器自动设置好，GET 方式的请求，参数会以 `?username=lisi&password=123` 方式追加到请求地址中，而 POST 方式的请求参数默认会被追加到请求体中，在 Ajax 中也同样有这样的设定。

GET 请求方式：

```js
// 2 设置请求方式、请求地址
let params = 'username=lisi&password=123'
xhr.open('get', 'http://localhost:3000/getDemo' + '?' + params)
// 3 发送请求
xhr.send()
```

GET 请求方式：

```js
// 2 设置请求方式、请求地址
let params = 'username=lisi&password=123'
xhr.open('post', 'http://localhost:3000/postDemo')
xhr.setRequestHeader(
  // POST 请求必须设置请求头
  'Content-Type',
  'application/x-www-form-urlencoded'
)
// 3 发送请求：在sen的中发送参数，POST的参数封装在请求体中
xhr.send(params)
```

### 5.2 参数传递方式

在上述案例中，请求的参数都是以字符串形式传递的：`"username=lisi&password=123"`，这种方式其实叫做 URL 编码格式。

通过设置请求头，请求参数的格式可以有多种传递方式：

```js
// URL 编码格式传递："username=lisi&password=123"
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded')

// JSON 格式传递: {username:"zs","password":"123"}，发送时必须转换为字符串
xhr.setRequestHeader('Content-Type', 'application/json')
xhr.send(JSON.stringify({ username: 'zs', password: '123' }))
```

注意：GET 请求只支持 URL 编码格式。

### 5.3 Ajax 错误处理

```js
xhr.onerror = function () {
  console.log('请求发生错误:', xhr.status)
}
```

### 5.4 Ajax 状态码

在创建 Ajax 对象，配置 Ajax 请求，发送请求，以及接收服务端响应数据过程中，每一个步骤都对应一个数值，即 Ajax 状态码。

```txt
0   请求未初始化（未调用open()）
1   请求已建立，但未发送（未调用send()）
2   请求已发送
3   请求正在处理中，此时一般已经接收到了一部分数据
4   响应完成
```

状态码的获取方式：`xhr.readyState`。状态码改变的监听事件为： `onreadystateChange`。

```js
xhr.onreadystatechange = function () {
  console.log(xhr.readyState) // 依次输出 1 2 3 4
}
xhr.send(params)
```

### 5.5 onload 与 onreadystatechange

```js
onload：              无需判断状态码，只调用一次，不兼容IE低版本
onreadystatechange：  需要判断状态码，会被调用多次，兼容IE低版本
```

## 六 一些开发问题

### 6.1 IE 中的请求缓存问题

IE8 中，请求的信息会被缓存下来，所以后续的星球会先从浏览器中直接获取结果，如果在这之间服务端出现了数据变化，Ajax 的获取到的数据却不会做响应变更。

解决方案：

```js
// 在请求地址的后面添加请求参数，每一次请求中的请求参数都不相同
xhr.open('get', 'http://www.demo.com?t=' + Math.random())
```
