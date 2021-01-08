今天来学习 Node.js 的 Web 框架 - Express

### 一. 一些基本概念

- Web 框架：服务器端框架(亦称 "web 应用框架") 使编写、维护和扩展 web 应用更加容易。它们提供工具和库来实现简单、常见的开发任务, 包括 路由处理, 数据库交互, 会话支持和用户验证, 格式化输出 (e.g. HTML, JSON, XML), 提高安全性应对网络攻击。
  Web 框架的主流思路是 MVC
  - M：Model 处理数据相关的逻辑
  - V：View 处理视图相关的逻辑（前后端分离后，这一部分基本由前端负责）
  - C：Controller： 处理业务和其他逻辑
- HTTP 协议:
  HTTP 协议不是三言两语可以说清楚的，HTTP 协议的内容很多有许多专门讲 HTTP 协议的书，也不知道为什么面试总喜欢问这些问题，短时间内根本说不完...
  所以这里就简单的说下，也为今后的面试梳理一下语言。

  1. HTTP 协议出现的目的
     HTTP 协议是为了共享知识而诞生的协议。

     > 欧洲核子研究组织的蒂姆·伯纳斯·李博士提出了一种能让远隔两地的研究者们共享知识的设想。

     > 最初设想的基本理念是:借助多文档之间相互关联形成的超文本 (HyperText)，连成可相互参阅的 WWW(World Wide Web，万维 网)。

     > 构建 WWW 的技术中就包括了：把 SGML(Standard Generalized Markup Language，标准通用标记语言)作为页面的文本标 记语言的 HTML(HyperText Markup Language，超文本标记语言); 作为文档传递协议的 HTTP ;指定文档所在地址的 URL(Uniform Resource Locator，统一资源定位符)。

  简单说就是：

  - HTML：用于描述文档
  - HTTP：用于传输文档
  - URL：用于定位文档

  2. HTTP 协议的作用
     HTTP 协议用于客户端和服务器端之间 的通信，客户端指的是请求资源的一方，响应资源的一方叫做客户端。
  3. HTTP 报文
     用于 HTTP 协议交互的信息被称为 HTTP 报文。
     请求端(客户端)的 HTTP 报文叫做请求报文，响应端(服务器端)的叫做响应报文。
     HTTP 报文本身是由多行(用 CR+LF 作换行符)数据构成的字符串文本。
     - 请求报文结构：
       ```
       请求行
       请求头
       空行（CR+LF）
       请求体（报文主体）
       ```
       - 请求行：包含用于请求的方法，请求 URI 和请求方法
       - 请求头：含有与所要获取的资源或者客户端自身相关的附加信息，比如：Host、User-Agent 等等
       - 空行：用于划分请求头和请求主体
       - 请求体：请求相关的数据
     - 响应报文结构：
       ```
       状态行
       响应头
       空行（CR+LF）
       响应体（报文主体）
       ```
       - 状态行：包含表明响应结果的状态码，原因短语和 HTTP 版本。
       - 响应头：含有与响应相关的附加信息，比如：Content-Type，Content-Length 等等
       - 空行：用于划分响应头和响应主体
       - 响应体：响应相关的数据
         可以看出不管是请求报文还是响应报文它们的结构都是一致的：
     ```
     报文头
     空行
     报文体
     ```
  4. HTTP 和 TCP/IP 的关系
     很多文章都会把 HTTP 协议和 TCP/IP 一起讲，我之前一直以为 TCP/IP 就是一个协议的名称，看了《图解 HTTP》之后才知道，互联网相关联的协议集合起来总称为 TCP/IP 协议族，简称 TCP/IP，TCP 和 IP 分别对应的是 TCP 协议和 IP 协议。
     它们之间的关系：
     ![](/madao.github.io/database/images/articles/node/express1/image.png)
     所以 HTTP 和 TCP/IP 的关系就是 HTTP 协议是 TCP/IP 协议族中的一个协议，位于 TCP/IP 协议族的应用层中，TCP/IP 协议族按层次分别分为: 应用层、传输层、网络层和数据链路层。
     至于三次握手和四次挥手什么的都是 TCP 协议的东西。

### 二. Express

Express 官方提供了脚手架工具 express-generator 可以快速搭建一个项目。
从创建好的项目中，可以看到使用了很多`app.use`方法，这个方法会将中间件添加到指定的路径上。
这里提到一个在 express 中很重要的概念“中间件”

#### 1. 中间件

中间件的意思就是一个请求从接收到响应请求的过程中执行的用于完成特定任务的函数，举个例子：

```
const express = require('express')
const app = express()
app.use('/xxx', (request, response, next) => {
  // 我就是中间件，我可以在请求/xxx这个路径的时候打印出当前的时间
  console.log(Date.now())
  next()
})
app.use('/xxx', (request, response, next) => {
  // 我也是中间件，我可以在请求/xxx这个路径的时候给响应结果中添加数据
  response.send('hello')
})
```

如果同一个路径有多个中间件，除了最后一个中间件可以不用调用 next 方法，其他的中间件都需要调用 next 以进入下一个中间件。
next 方法可以传入一个参数作为错误的信息，比如：

```
app.use('/xxx', (request, response, next) => {
  next('出错了')
})
app.use((error, request, response, next) => {
  response.send(error) // 响应结果就是'出错了'
})
```

记住错误处理需要把四个参数都写上
next 还有一种特别的用法`next('route')`，它的用法如下：

```
app.use((request, response, next) => {
  next('route')
}, (request, response, next) => {
  console.log(2)
  next()
})
```

当出现这传入多个函数的情况时，next('route')会跳过剩余的函数

#### 2. router

看完中间件之后发现路由功能可以使用中间件实现，但是 express 提供了一个 router 方法，它的用法我的[另一篇笔记](https://greedywhale.github.io/madao.github.io/#/article/node/Express)总结过，这里就不重复了
