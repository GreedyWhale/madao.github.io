### 一. Node.js 是什么

Node.js 不是编程语言，不是一个Web框架，Node.js是一个平台，它可以让JavaScript调用操作系统提供的接口，让JavaScript也能开发后端应用。

Node.js 技术架构图：

![](/madao.github.io/database/images/articles/node/basis/image.png)

- Node.js API: Node.js内置的标准库
- Node.js bindings: js代码和c/c++代码沟通的桥梁，比如在Node.js 中可以读写文件，比如：
`fs.readFile()`，js代码并没有读写文件的能力，所以调用`fs.readFile()`实际上是c++代码去完成这个操作，从js代码调用，然后c++代码响应并执行，中间的通道就是bindings，其实在浏览器环境只能够也存在bindings，比如：`document.querySelector()`，它并不是js提供的功能，它由浏览器提供。
- c/c++ addons：c++插件，Node.js 提供插件功能，可以让原生的c++代码编译成`.node`文件，这样在Node.js中，可以通过require直接引用
- v8：大名鼎鼎的js引擎，在Node.js中提供js的运行环境
- libuv：一个跨平台的异步I/O库。
- c-ares、http_parser...：一些使用c++编写的库（目前还不是很了解这些库）





