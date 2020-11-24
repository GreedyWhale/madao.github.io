今天学习的是Node.js中的HTTP模块，我之前有总结过相关知识，但是忘的差不多了，重新来一遍吧...

### 零. 工具推荐

1. [node-dev](https://www.npmjs.com/package/node-dev)

    当文件改变后，自动重启node.js

2. [ts-node](https://www.npmjs.com/package/ts-node)

    让ts代码直接在Node中运行

3. [ts-node-dev](https://www.npmjs.com/package/ts-node-dev)

    这个工具是上面两个工具的结合版，让Node.js支持Typescript和热更新。

4. curl

    curl是一个命令行工具，可以用它来发送请求，用法可以参考[curl 的用法指南](https://www.ruanyifeng.com/blog/2019/09/curl-reference.html)

### 一. server

当我们在浏览器上访问各种各样的网页时，实际上就是向服务器发送请求，服务器返回对应的文件给用户，这种请求叫HTTP请求，因为它使用的传输协议是HTTP，在Node.js中也有对应的模块来处理HTTP请求，这个模块就叫做http，首先从`http.createServer`说起：

`http.createServer` 返回一个可以处理HTTP请求对象，它是http.Server类的实例，而http.Server类又继承自net.Server 类，通过文档可以找到这个两个方法：

1. http.createServer().on('request')

    监听请求事件，每当有请求时触发

2. http.createServer().listen()

    启动一个服务器来监听连接，里面可以传入端口号等配置。

现在就可以通过这两个方法启动一个HTTP服务器：

```
import { createServer } from 'http'

const server = createServer()

server.on('request', () => {
  console.log('请求来了')
})

server.listen(1234)
```

通过ts-node-dev执行这个代码，然后用curl去请求`http://localhost:1234/`

结果：
![](/madao.github.io/database/images/articles/node/http2/image.png)

这样就能知道请求来了，但是作为服务器的一方并没有响应，那么接下来看看如何响应一个请求

### 二. request和response

在request事件的监听函数中，接受两个参数一个是request一个是response，通过名字就知道了，request是请求相关信息，response是响应相关的信息，Node的type文件是这样写的

![](/madao.github.io/database/images/articles/node/http2/image1.png)

它使用的是any...，所以还是得看文档才行。

通过文档可以知道response可以通过end或者write向发出请求的一方返回数据：

```
import { createServer } from 'http'

const server = createServer()

server.on('request', (request, response) => {
  response.end('hello')
})

server.listen(1234)
```

![](/madao.github.io/database/images/articles/node/http2/image1.png)


```
import { createServer } from 'http'

const server = createServer()

server.on('request', (request, response) => {
  response.write('hello\n')
  response.write('world')

  response.end()
})

server.listen(1234)
```

![](/madao.github.io/database/images/articles/node/http2/image2.png)

write可以写入多次，调用end会将write写入的所有数据返回。


![](/madao.github.io/database/images/articles/node/http2/image3.png)


### 三. 根据请求url返回不同的文件

要实现这一需求需要做：

1. 获取到用户请求的url

    ```
    import { createServer, IncomingMessage, ServerResponse } from 'http'

    const server = createServer()

    server.on('request', (request: IncomingMessage, response: ServerResponse) => {
      console.log(request.url)
      response.end()
    })

    server.listen(1234)
    ```

    ![](/madao.github.io/database/images/articles/node/http2/image4.png)

2. 根据获取到的url，获取到对应的资源

    虽然上一步，成功的拿到了url，但是有一个问题，如果用户在请求上面增加了参数就会变成这样：

    ![](/madao.github.io/database/images/articles/node/http2/image5.png)

    如果不想要这个参数，Node.js提供了一个url模块，里面的parse可以很方便的进行解析：

    ```
    import { createServer, IncomingMessage, ServerResponse } from 'http'
    import { parse } from 'url'

    const server = createServer()

    server.on('request', (request: IncomingMessage, response: ServerResponse) => {
      console.log('请求的url是：', parse(request.url))
      response.end()
    })

    server.listen(1234)
    ```

    ![](/madao.github.io/database/images/articles/node/http2/image6.png)

    接下来就很方便了，通过url.parse解析获取到pathname，然后通过path模块的join方法拼接成资源路径，然后找到匹配的资源返回即可：

    ```
    import { createServer, IncomingMessage, ServerResponse } from 'http'
    import { parse } from 'url'
    import { join } from 'path'
    import { readFile } from 'fs'

    const publicPath = join(__dirname, 'public')
    const server = createServer()

    server.on('request', (request: IncomingMessage, response: ServerResponse) => {
      const { pathname } = parse(request.url)
      readFile(join(publicPath, pathname), (error, data) => {
        if (error) {
          console.error(error)
          response.statusCode = 404
          response.end()
          return
        }
        response.end(data.toString())
      })
    })

    server.listen(1234)
    ```

    新建public目录：

    ![](/madao.github.io/database/images/articles/node/http2/image7.png)

    index.html：

    ```
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
      <link rel="stylesheet" href="/css/style.css">
    </head>
    <body>
      hello world
      <script src="/js/index.js"></script>
    </body>
    </html>
    ```

    打开浏览器，输入`http://localhost:1234/index.html?q=123123`

    可以看到资源都正确的拿到了：

    ![](/madao.github.io/database/images/articles/node/http2/image8.png)

    接下来优化一下，因为上面代码错误处理都是统一的使用404，并不是所有的错误都是资源未找到的，所以优化一下错误处理：

    ```
    server.on('request', (request: IncomingMessage, response: ServerResponse) => {
      const { pathname } = parse(request.url)
      readFile(join(publicPath, pathname), (error, data) => {
        if (error) {
          switch(error.code) {
            case 'ENOENT': // 文件不存在
              response.statusCode = 404
              break
            case 'EISDIR': // 请求资源是一个目录
              response.statusCode = 403
              break
            default:
              response.statusCode = 500
              break
          }
          response.end()
          return
        }
        response.end(data.toString())
      })
    })
    ```

    还有一个优化就是Content-Type的设置，可以根据请求的文件的扩展名来设置不同的Content-Type

    ```
    import { join, extname } from 'path'

    const setContentType = (filename: string): string => {
      const extensionName = extname(filename).replace(/^\./, '')
      const contentTypeMap = {
        js: 'text/javascript',
        css: 'text/css',
        html: 'text/html'
      }
      return contentTypeMap[extensionName] || ''
    }

    server.on('request', (request: IncomingMessage, response: ServerResponse) => {
      const { pathname } = parse(request.url)
      readFile(join(publicPath, pathname), (error, data) => {
        // ....省略之前的代码
        response.setHeader('Content-type', setContentType(pathname))
        response.end(data.toString())
      })
    })
    ```





### 四. 处理请求参数

请求参数可以写在url上，也可以放在请求报文的请求体中，也就是body中，可以通过如下方法获取到这两种请求参数：

1. 在url上携带请求参数

    在之前的例子中也能看到，通过url.parse解析request.url可以很方便的获取到请求参数

2. 获取在请求体中的参数

    获取在请求体中的数据，需要监听data事件和end事件：

    data事件：当接收到数据的时触发该事件。data 参数是一个 Buffer 或 String。

    end事件：当 socket 的另一端发送一个 FIN 包的时候触发，从而结束 socket 的可读端。（个人理解为数据传输完毕）

    ```
    let chunks = []
    request.on('data', (chunk) => {
      chunks.push(chunk)
    })
    request.on('end', () => {
      console.log(Buffer.concat(chunks))
    })
    ```

    ![](/madao.github.io/database/images/articles/node/http2/image9.png)

    但是这里我有个疑问，当数据量过大的时候如何处理呢，就算是每次接收少量的数据，用于拼接成完整数据的变量还是在内存中，难道需要变接收边存进硬盘吗？

### 五. 给响应头添加缓存控制

合理的利用HTTP缓存可以极大缩短页面打开速度，HTTP缓存又可以分为协商缓存和强制缓存：

**协商缓存**：浏览器去问服务器，当前缓存是否过期，如果过期就返回新资源，如果没过期就继续使用本地缓存，控制协商缓存的字段是：
- Last-Modified，If-Modified-Since
    - Last-Modified：文件最后一次修改的日期
    - If-Modified-Since：服务器上一次返回的Last-Modified值
- ETag、If-None-Match
    - ETag：资源唯一标识，只要资源发生变化，ETag就会改变
    - If-None-Match：服务器上一次返回的ETag

**强制缓存**：浏览器不去询问服务器，直接使用本地缓存，控制强制缓存的字段是：
- Expires：资源过期时间，在这个时间之前表示本地缓存都是有效的，比如：`Expires=Tue, 24 Nov 2020 02:44:34 GMT`
- Cache-Control：Cache-Control中可以设置很多值，其中控制资源过期时间的字段是max-age，max-age用于设置缓存存储的最大周期，超过这个时间缓存被认为过期，它是一个相对时间，相对于第一次请求。单位是秒，比如：`max-age=31536000`

使用setHeader或者writeHeader进行设置：

`response.setHeader('Cache-Control', 'public, max-age=31536000')`

```
response.writeHead(200, {
  'Cache-Control': 'public, max-age=31536000'
})
```

