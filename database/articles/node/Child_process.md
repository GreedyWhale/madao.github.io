今天学习 Node.js 中的 child_process 模块，学习这个模块需要进程和线程相关的知识，之前学习 python 的时候有接触过，但是基本忘光了...所以再学一遍吧。

### 一. 进程

感觉进程的定义很难描述清楚，看了一些文章，大多数都是使用一些例子来说明进程是什么，看完之后大概也能明白进程是什么，但是当我试图去解释进程是什么的时候，又很难说出来，比如现在...

还是决定使用维基百科中的解释，虽然它并不好理解：

> 进程（英语：process），是指计算机中已运行的程序。进程曾经是分时系统的基本运作单位。在面向进程设计的系统（如早期的 UNIX，Linux 2.4 及更早的版本）中，进程是程序的基本执行实体；在面向线程设计的系统（如当代多数操作系统、Linux 2.6 及更新的版本）中，进程本身不是基本运行单位，而是线程的容器。

从这段描述中可以看出，进程经历了三个阶段：

1. 分时系统的基本运作单位
2. 程序的基本执行实体
3. 线程的容器

这三个阶段都引用了一些其他概念去解释进程是什么，尴尬的是我其实对“其他概念”也是不了解，比如：分时系统，执行实体，线程...

根据维基百科中所说的，当代多数操作系统都是面向线程设计的系统，所以进程目前的定义应该符合“线程的容器”这一阶段，所以先记住它。

根据自己的理解总结下进程的特点：

当用户打开一个应用程序的时候，就会创建进程，可通过操作系统的任务管理器看到，同一程序可产生多个进程，一个进程也可以创建另一个进程，创建进程的进程叫做父进程，被创建的进程叫做子进程，进程与进程之间是相互隔离的，它们不会共享内存空间

- 应用程序被执行的时候会产生进程
- 同一个应用程序可以产生多个进程
- 进程本身也可以创建进程，父进程 -> 子进程
- 进程之间相互隔离，不共享内存空间

### 二. 线程

线程包含在进程中，一个进程中可以包含多个线程，线程是进程中的实际运作单位，比如我的程序既可以聊天也能听歌，那么可能聊天功能就是由一个线程负责，听歌功能由另一个线程负责，同一个进程中的线程共享资源。

这里留一个疑问？为什么需要线程（暂时还没梳理清楚）

如果上面说的还不能理解，那么只能用阮一峰大佬文章[进程与线程的一个简单解释](https://www.ruanyifeng.com/blog/2013/04/processes_and_threads.html)里的比喻了:

CPU(工厂) -> 车间（进程）-> 车间里的工人（线程）

### 三. child_process

child_process 模块提供了创建子进程的功能，常用的 API 有：

1. `child_process.exec(command[, options][, callback])`

   child_process.exec 用于执行 bash 命令

   - 可通过回调函数获取执行结果
   - 可通过监听事件获取执行结果
   - 可通过`util.promisify()`将其 promise 化，避免回调地狱

   例子：

   ```
   const { exec } = require('child_process')
   const { promisify } = require('util')

   exec('ls -lh', (error, stdout, stderr) => {
     if (error) {
       throw error
     }
     console.log('通过回调获取到的结果')
     console.log(stdout)
   })

   const myProcess = exec('ls -lh')
   myProcess.stdout.on('data', (chunk) => {
     console.log('通过监听事件获取到的结果')
     console.log(chunk.toString())
   })

   promisify(exec)('ls -lh')
     .then(result => {
       console.log('通过promise化获取到的结果')
       console.log(result)
     })
   ```

   这个方法需要注意的就是，命令和命令的参数都是写在一起的，会有一定的安全风险，比如：

   ```
   exec('ls -lh; pwd', (error, stdout, stderr) => {
     console.log(stdout)
   })
   ```

   还需要注意的一点是，exec 方法可以设置 maxBuffer，maxBuffer 指的是 stdout 或 stderr 上允许的最大字节数，如果超过这个值，则子进程会被终止，默认是 1024 \* 1024（1MB）

2. `child_process.execFile(file[, args][, options][, callback])`

   execFile 和 exec 的区别就是，execFile 将传入命令的参数作为第二个参数以数组的形式传入，这样就避免了上面说的安全风险

   ```
   // 这样执行会报错
    execFile('ls', ['-lh', ';pwd'], (error, stdout, stderr) => {
      if(error) { throw error }
      console.log(stdout)
    })
   ```

   但是这个方法同样有 maxBuffer 的限制

3. `child_process.spawn(command[, args][, options])`

   child_process.spawn 的功能也是执行传入的命令，但是它不能通过回调函数的形式获取到结果，它只能通过事件监听的形式获取到，他和 exec、execFile 不同的一点是，他没有 maxBuffer 的限制。

   可以看出 exec、execFile、spawn 的功能基本一样...

   推荐使用优先级：`spawn > execFile > exec`

   不知道 Node.js 为啥出这么多相似的 API

4. `child_process.fork(modulePath[, args][, options])`

   child_process.fork 的用法和 spawn 一样，不同点在于 child_process.fork 会创建一个 Node 进程，

   ```
   fork('./child.js')
   ```

   相当于

   ```
   spawn('node', ['./child.js'])
   ```

   除此之外，fork 会多出一个 message 事件和 send 方法，message 事件用于接收数据，send 方法用于发送数据，用法如下：

   main.js

   ```
   const { fork } = require('child_process')

   const process1 = fork('./child.js')
   process1.on('message', (message) => {
     console.log('从子进程获取到的数据:')
     console.log(message)
   })
   process1.send('今天的风儿好喧嚣啊！')
   ```

   child.js

   ```
   process.on('message', (message) => {
     console.log('从父进程获取到的数据:')
     console.log(message)
   })

   process.send('不得了了，隔壁超市薯片半价了，快去买！')
   ```
