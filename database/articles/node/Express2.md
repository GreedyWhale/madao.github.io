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
     ![](/madao.github.io/database/images/articles/node/express2/image.png)
     所以 HTTP 和 TCP/IP 的关系就是 HTTP 协议是 TCP/IP 协议族中的一个协议，位于 TCP/IP 协议族的应用层中，TCP/IP 协议族按层次分别分为: 应用层、传输层、网络层和数据链路层。
     至于三次握手和四次挥手什么的都是 TCP 协议的东西。

### 二. Express

express 是一个 Node.js Web 应用程序框架，它的核心是**中间件**，如下图：

![](/madao.github.io/database/images/articles/node/express2/image1.png)

中间件函数可以执行以下任务：

- 执行任何代码。
- 对请求和响应对象进行更改。
- 结束请求/响应循环。
- 调用堆栈中的下一个中间件。

从图中可以看出 express 的中间件模型是呈一条直线的，这里需要记一下，这也是它和另外一个框架 Koa 最大的不同。

举个例子：

```
const express = require('express')
const app = express()

app.use('/hello', (request, response, next) => {
  console.log('我是中间件1')
  response.write('1\n')
  next()
})

app.use('/hello', (request, response, next) => {
  console.log('我是中间件2')
  response.write('2\n')
  next()
})

app.use('/hello', (request, response, next) => {
  console.log('我是中间件3')
  response.write('3\n')
  response.send()
})

app.listen('1234', () => {
  console.log('启动成功')
})
```

使用 node 去执行上面代码就会启动一个本地服务器，访问`http://localhost:1234/hello`，就会得到 123。

官网也对编写中间件有很好的说明：
![](/madao.github.io/database/images/articles/node/express2/image2.png)

- #### Express 的 API

  Express 的 API 分为 5 大类：

  - express
  - Application
  - Request
  - Response
  - Router

  下面分别来看看：

  - express.xxx

    - express.json()

      `express.json`将请求体中的数据以 json 的格式解析。

      ```
      app.use(express.json())
      app.use('/hello', (req, res, next) => {
        const { body: { name } } = req
        res.send(`你好! ${name}`)
      })
      ```

      如果不用`express.json`，那么就只能通过监听`data`事件来获取请求体中的数据了：

      ```
      app.use('/hello', (req, res, next) => {
        let requestBody = ''
        req.on('data', (chunk) => {
          requestBody += chunk.toString()
        })
        req.on('end', () => {
          res.send(requestBody)
        })
      })
      ```

    - express.raw()

      `express.raw`将请求体中的数据变成 Buffer（我理解的 Buffer 就是在内存中的二进制数据）形式，

    - express.Router()

      创建一个新的 Router 对象，例子：

      ```
      const app = express()
      const childRouter = express.Router()

      app.use('/', (req, res, next) => {
        res.write('我来组成头部')
        next()
      })

      childRouter.use('/', (req, res, next) => {
        res.write('我来组成身体')
        res.end()
      })

      app.use('/childRouter', childRouter)
      ```

    - express.static()

      `express.static`用于提供静态文件，比如你的所有图片、css 文件、js 文件等静态资源都存放在项目目录下的`public`目录下，只需要设置：`app.use(express.static('public'))`，然后资源的请求就会自动的去 public 目录下寻找：`http://localhost:8080/images.png` 相当于 `http://localhost:8080/public/images.png`

    - express.text()

      和`express.json`类似，用于解析以`text/plain`格式提交的请求数据。

    - express.urlencoded()

      用于解析以`application/x-www-form-urlencoded`格式提交的请求数据，x-www-form-urlencoded 格式是这样的：

      `name=xx&age=123`

  - app.xxx

    app 提供的 API 有点多，一共有 22 个，常用的有：`app.use`、`app.get`、`app.set`、`app.post, app.put...`、`app.render`

    - app.use

      将中间挂载在指定的路径上，例子：

      ```
      // 正常的按照顺序执行的中间件
      app.use('/', (req, res, next) => {
        res.write(1)
        next()
      })
      app.use('/', (req, res, next) => {
        res.write(2)
        next()
      })
      app.use('/', (req, res, next) => {
        res.write(3)
        res.end()
      })

      // 跳去错误处理中间件
      app.use('/', (req, res, next) => {
        next('出错了')
      })
      // 错误处理中间件
      app.use((err, req, res, next) => {
        res.status(500).send(err) // 出错了
      })

      // 忽略中间件
      app.use('/', (req, res, next) => {
        next('route')
      }, (req, res, next) => {
        console.log('我不会被执行，我被忽略了')
      })
      ```

    - app.get，app.set

      app.get 有两种用法，一种是和 app.set 配合使用，例如：

      ```
      app.use('/', (req, res, next) => {
        app.set('name', 'jack')
        next()
      })

      app.use('/', (req, res, next) => {
        const name = app.get('name') // jack
        res.send(name)
      })
      ```

      另一种则是作为 use 的语法糖使用，例子：

      ```
      app.get('/name', (req, res, next) => {
        res.send('123')
      })
      ```

      等于

      ```
      app.use('/name', (req, res, next) => {
        if (req.method === 'GET') {
          res.send('123')
        }
      })
      ```

    - app.put、app.post、app.delete...

      这类 API 都是 app.use 的语法糖，就跟 app.get 一样，put、post、delete 这些对应的都是 http 的请求类型

    - app.render

      app.render 返回视图的 HTML 字符串，例子：

      ```
      app.set('views', './views')
      app.set('view engine', 'pug')

      app.render('index', (err: any, html: string) => {
        console.log(html)
      })
      ```

      `app.set('views', './views')`是用来设置视图的模版文件的位置，`app.set('view engine', 'pug')`用来设置解析视图模版的引擎：

      ```
      app.set('views', './views')
      app.set('view engine', 'pug')
      app.get('/', (req, res) => {
        app.render('index', (err: any, html: string) => {
          res.send(html)
        })
      })
      ```

      等于

      ```
      app.set('views', './views')
      app.set('view engine', 'pug')

      app.get('/', (req, res) => {
        res.render('index')
      })
      ```

  - request.xxx

    request 一共提供了 28 个 API，大部分根据 API 的名字就能猜到是用来做什么的，有一个需要注意下：`request.range()`

    - request.range

      分块下载，专业解释[MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Range_requests)，简单说就是发出多个请求去下载一个资源，每个请求只负责下载该资源的一部分。

      request.range 接收一个 size 参数，表示该资源的最大值，当请求头中有：`Range: bytes=0-50, 100-150`时，调用 request.range 就会返回：
      `[ { start: 0, end: 50 }, { start: 100, end: 150 }, type: 'bytes' ]`

      根据这个结果再配合 Node.js 中的流就可以实现分块下载的需求了。

  - Response.xxx

    这一部分 API 感觉也没有特别需要注意的，基本上文档都说的比较清楚了，所以这里就不再赘述了

  - Router.xxx

    这是一个轻量级的 app，用法和 app 基本一样，也没啥好说的。

  总结下：

  - express.xxx: 内置的中间件
  - app.xxx: 应用级别的设置
  - request.xxx: 请求相关
  - response.xxx: 响应相关
  - router.xxx: 路由级别的设置

大概看完 express 的文档后，感觉就是用好中间件就用好了 express。
