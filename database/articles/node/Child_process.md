今天学习 Node.js 中的 child_process 模块，学习这个模块需要进程和线程相关的知识，之前学习 python 的时候有接触过，但是基本忘光了...所以再学一遍吧。

### 一. 进程

#### 1. 定义

我对进程的理解是这样的：

_进程就是正在执行的程序或者程序的一次执行过程。_

专业一点的定义就是：一个执行中程序的实例

程序是存储在计算机上用于完成特定任务的代码，计算机执行某个程序的时候，会将该程序的代码以二进制的形式加载到内存中，除了代码之外，程序根据自己需要完成的任务还需要一些其他的资源，比如打印程序需要打印机资源等等，程序执行时所需要的系统资源，数据等等的集合就是进程，系统资源以进程为单位分配，所以还有一句常常听到的话“进程是资源分配的基本单位”

#### 2. 为什么需要进程

早期的操作系统一次只能执行一个程序，而 CPU 的效率又远远超高计算机其他资源的效率，比如 IO 操作，当程序进行 IO 操作的时候，CPU 是空闲状态，为了提高 CPU 的利用率，就需要一种可以让多个程序同时进入内存并执行的技术，于是进程出现了，CPU 在各个进程之间来回快速切换达到多个程序并行的效果，单核的 CPU 在同一时间只能运行一个线程。

#### 3. 进程的特点

- 每个进程都拥有独立的内存地址空间。
- 进程之间不共享数据
- 一个进程可以创建其他进程，所以一个程序可以有多个进程

### 二. 线程

#### 1. 定义

> 线程（英语：thread）是操作系统能够进行运算调度的最小单位。--- 维基百科

调度的意思就是操作系统将 CPU 的使用权交给某个进程或线程让 CPU 去运行它。

我理解的线程是“轻量级的进程”，轻量级指的是操作线程的开销要低于操作进程的开销，比如：创建、切换、销毁这些操作。

线程包含在进程中，线程和进程之间的关系就像外包公司和程序员，公司接活并提供给程序员开发所需的环境，然后程序员进行开发。

#### 2. 为什么需要线程

进程的出现让操作系统可以并发的执行程序，线程可以让程序并发的执行任务

并发和并行的区别：

![](/madao.github.io/database/images/articles/node/child_process/image.png)

举个例子：

加入某个程序可以一边输入一边自动保存，如果没有线程，那么在单核 CPU 的情况下，软件自动保存的时候，用户就无法输入，引入了线程后，自动保存和输入就可以交给该程序下的不同线程去做，当该程序的进程获得了 CPU 的使用权后，就可以在这些线程中快速的切换并发的执行任务。

#### 3. 线程的特点

- 操作系统能够进行运算调度的最小单位
- 一个进程中至少有一个线程，可以拥有多个线程
- 一个进程中的所有线程共享该进程的所有资源
- 线程的开销低于进程

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
