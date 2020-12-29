牙白牙白，今年就要结束了，并没有达成之前立的 flag，那么明年我要...

今天学习 Node.js 中的 Stream 模块，Stream 翻译过来就是流的意思，从字面意思不太好理解，文档上说：

> 流（stream）是 Node.js 中处理流式数据的抽象接口。

我目前的理解就是，如果把存储数据的地方看作一个水池，要填满这个水池需要一个水龙头也就是数据源，还需要一根管道将水龙头和水池连接起来，水就是数据了，可以一次性将水池填满，也可以多次打开水龙头将水池填满，流使用多次传输，每次传输一部分数据的模式，下面看看这样做的好处。

### 一. 如果没有 Stream

假如服务器上有一个比较大的文件，用户通过 http 请求可以拿到这个文件：

```
const fs = require('fs')
const http = require('http')
// 生成一个大文件
const createBigFile = () => {
  let i = 10000000
  const bigFile = fs.createWriteStream('./bigFile.txt')
  while(i > 0) {
    bigFile.write(`这是第${i}行\n`)
    i -= 1
  }
  bigFile.end()
  console.log('大文件生成完毕')
}

createBigFile()
const server = http.createServer()

server.on('request', (request, response) => {
  fs.readFile('./bigFile.txt', (error, data) => {
    if (error) throw error
    response.end(data)
  })
})

server.listen(12345)
```

通过上面的代码，可以生成一个将近 200M 的 txt 格式的文件，并且会开启一个服务。

我这台机器目前 node 使用的内存是：

![](/madao.github.io/database/images/articles/node/stream/image.png)

接下来用 curl 去请求一下这个文件：

`curl http://localhost:12345`

![](/madao.github.io/database/images/articles/node/stream/image1.png)

可以看到 node 使用的内存一下子就变的很高，显而易见这种处理不满足需求，只要请求一多内存就不够用了。

现在改用流的形式：

```
const server = http.createServer()

server.on('request', (request, response) => {
  const stream = fs.createReadStream('./bigFile.txt')
  stream.pipe(response)
})

server.listen(12345)
```

初始占用内存：

![](/madao.github.io/database/images/articles/node/stream/image.png)

用 curl 去请求这个文件：

`curl http://localhost:12345`

![](/madao.github.io/database/images/articles/node/stream/image2.png)

这是在请求过程中占用最多一次，基本上不会超过 30M，使用了 stream 后，极大的降低了 Node.js 在处理大量数据时所占用的内存。

### 二. pipe

上一节中的例子中有使用到一个方法`pipe`，`pipe`翻译过来时管道的意思，它的作用也正如它的字面意思一样，pipe 可以将一个可读流与可写流连接起来，将可读流里的数据传入可写流中，比如：`stream1.pipe(stream2)`。

pipe 还支持链式调用：

```
a.pipe(b)
 .pipe(c)
 .pipe(d)
```

相当于：

```
a.pipe(b)
b.pipe(c)
c.pipe(d)
```

pipe 方法也可以通过事件实现：

```
readable.on('data', (chunk) => {
  writable.write(chunk)
})

readable.on('end', () => {
  writable.end()
})
```

Node.js 中的许多模块都实现了流接口：

| 可读流(Readable Streams)        | 可写流(Writable Streams)       |
| ------------------------------- | ------------------------------ |
| HTTP responses, on the client   | HTTP requests, on the client   |
| HTTP requests, on the server    | HTTP responses, on the server  |
| fs read streams                 | fs write streams               |
| zlib streams                    | zlib streams                   |
| crypto streams                  | crypto streams                 |
| TCP sockets                     | TCP sockets                    |
| child process stdout and stderr | child process stdin            |
| process.stdin                   | process.stdout, process.stderr |

从上表中可以看出服务端的 responses 是在可写流那一列中，那么通过`fs.createReadStream`创建的可读流就能用`pipe`和 response 连接起来。

除了使用`pipe`进行连接，也可以用事件：

```
readable.on("data", chunk => {
  writable.write(chunk);
});

readable.on("end", () => {
  writable.end();
});
```

流中的一小块数据一般成为`chunk`

### 三. 流的分类

Node.js 中流可以分为四种：

- Readable：可读流
- Writable：可写流
- Duplex：双工流
- Transform：转换流

#### 1. Readable

Readable 是可读流，它的特点就是可以多次读取流中的数据，可以利用 Readable 类实现一个自定义的可读流，例子：

```
const { Readable } = require('stream')

const producer = new Readable()

producer.push('其实长谷川先生只是运气不好罢了\n')
producer.push('MADAO总会开花的\n')

producer.push(null) // 没有更多数据了

producer.pipe(process.stdout)
```

结果：

![](/madao.github.io/database/images/articles/node/stream/image3.png)

这种情况是实现在可读流中准备好数据，然后在 pipe 到可写流中，还有一种方式是，在读的时候再准备数据：

```
const { Readable } = require('stream')

const producer = new Readable({
  read (size) {
    this.push(String.fromCharCode(this.charCode++))
    if (this.charCode > 70) {
      this.push(null)
    }
  }
})

producer.charCode = 65

producer.pipe(process.stdout)
```

只需要在创建 Readable 的实例的时候传入一个含有 read 方法的对象，那么该实例在每次读取数据的时候就会调用传入的 read 方法，read 方法接受一个 size 参数，size 指定每次读取的数据大小，size 必须小于或等于 1 GiB

可读流有两种状态：静止态（paused）和 流动态（flowing）

- 可读流默认处于静止态
- 当可读流被添加了`data`事件，它就自动变为流动态
- 当可读流的`data`事件被移除，它就变回静止态
- 这两种状态可通过 resume() 和 pause()进行手动切换
- 当可读流处于静止态时，可以调用 read()方法读取数据，当可读流处于流动态时，数据会一直流动，必须监听事件处理数据，否则数据会丢失。

#### 2. writable

writable 是可写流，它的特点就是可以多次向流中写入数据，同样也可以实现自定义的可写流，例子：

```
const { Writable } = require('stream')

const consumer = new Writable({
  write (chunk, encoding, callback) {
    console.log(`你输入的是：${chunk.toString()}`)
    callback()
  }
})

process.stdin.pipe(consumer)
```

效果：

![](/madao.github.io/database/images/articles/node/stream/image4.png)

#### 3. Duplex

Duplex 指的就是既可以读也可以写的流，但是这里的读和写都是互不干扰的，完全独立运行，例子：

```
const { Duplex } = require('stream')

const duplexStream = new Duplex({
  write (chunk, encoding, callback) {
    console.log(`你输入的是：${chunk.toString()}`)
    callback()
  },
  read (size) {
    this.push(`${String.fromCharCode(this.charCode++)}\n`)
    if (this.charCode > 70) {
      this.push(null)
    }
  }
})

duplexStream.charCode = 65

process.stdin
  .pipe(duplexStream)
  .pipe(process.stdout)
```

效果：

![](/madao.github.io/database/images/articles/node/stream/image5.png)

当可读流 pipe 一个可写流的时候，就相当于给可读流添加了 data 事件，可读流的状态就会变成流动态，所以会先输出: `ABCDEF`

#### 4. Transform

Transform 是转换流，它也同时支持读写，它可以将写入的数据进行转换然后再输出，概念有点像 Webpack 中的 loader，比如 babel-loader，可以用它把使用新语法的 js 代码转译成使用旧语法的 js 代码，例子：

```
const { Transform } = require('stream')

const transformStream = new Transform({
  transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase())
    callback()
  }
})

process.stdin
  .pipe(transformStream)
  .pipe(process.stdout)
```

效果：

![](/madao.github.io/database/images/articles/node/stream/image6.png)

这个转换流可以将输入的小写字母转换为大写字母再输入

### 四. stream 模块中一些常用的方法和事件

|      | Readable Streams                                                                                     | Writable Streams                                                  |
| ---- | ---------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| 事件 | data, end, error, close, readable                                                                    | drain, finish, error, close, pipe, unpipe                         |
| 方法 | pipe(), unpipe(), wrap(), destroy(), read(), unshift(), resume(), pause(), isPaused(), setEncoding() | write(), destroy(), end(), cork(), uncork(), setDefaultEncoding() |

这里主要说一下 drain 事件，有这么一种情况， 当 Readable 传输给 Writable 的速度远大于它接受和处理的速度，简单说就是读的太快，写的太慢，这时候就会有数据积压的情况出现，如果调用 stream.write(chunk) 返回 false 后就不要在调用 write 方法了，这时需要监听 drain 事件，当 drain 事件触发后，就可以继续写入了。
