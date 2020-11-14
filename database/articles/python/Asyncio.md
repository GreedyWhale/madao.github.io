### 一. 协程

我对协程的理解就是在单线程中执行函数 A，可以中断函数 A 的执行，切换去执行其他函数，在合适的时候（由程序自己控制）在切换回 A 继续执行，达到类似多线程的效果。

这样做的好处是：

1. 没有线程的切换，所以不会有线程切换的开销。
2. 因为只有一个线程，所以修改共享的变量不需要锁
3. 协程之间的切换由程序控制而不是操作系统控制

### 二. 协程的效率

- 计算密集型

  在计算密集型的程序中，协程的效率并不是很高，反而会变低：

  ```
  from threading import Thread
  import time
  import queue


  def work(q):
      res = 0
      for i in range(1000000):
          res += i ** 2
      q.put(res)


  def mutil_thread():
      res = 0
      threads = []
      q = queue.Queue()
      for i in range(2):
          t = Thread(target=work, args=(q,))
          t.start()
          threads.append(t)

      [thread.join() for thread in threads]
      while not q.empty():
          res += q.get()
      print(res)


  def coroutine_consumer():
      n = 0
      while True:
          m = yield n
          if not isinstance(m, int):
              return
          n += m ** 2


  def coroutine_producer(consumer):
      consumer.send(None)
      for i in range(2):
          for i in range(1000000):
              res = consumer.send(i)
      consumer.close()
      print(res)


  def normal():
      res = 0
      for i in range(2):
          for j in range(1000000):
              res += j ** 2
      print(res)


  t1 = time.time()
  normal()
  print('单线程：%fs' % (time.time() - t1))
  t2 = time.time()
  mutil_thread()
  print('多线程：%fs' % (time.time() - t2))
  t3 = time.time()
  coroutine_producer(coroutine_consumer())
  print('协程：%fs' % (time.time() - t3))
  ```

  结果：

  ![](/madao.github.io/database/images/articles/python/asyncio/image.png)

  可以看到协程是最慢的

- IO 密集型

  ```
  import time
  import asyncio
  import threading


  def work(t):
      print('task %d start' % t)
      time.sleep(t)   # 想象这是一个费时的io操作
      print('task %d end' % t)


  def normal():
      [work(i) for i in range(1, 3)]


  def multi_thread():
      threads = []
      for i in range(1, 3):
          t = threading.Thread(target=work, args=(i,))
          t.start()
          threads.append(t)
      [thread.join() for thread in threads]


  async def work_coroutine(t):
      print('task %d start' % t)
      await asyncio.sleep(t)  # 想象这是一个费时的io操作
      print('task %d end' % t)


  async def main(loop):
      tasks = [loop.create_task(work_coroutine(t)) for t in range(1, 3)]
      await asyncio.wait(tasks)


  t1 = time.time()
  normal()
  print('单线程：%fs' % (time.time() - t1))
  t2 = time.time()
  loop = asyncio.get_event_loop()
  loop.run_until_complete(main(loop))
  loop.close()
  print('协程：%fs' % (time.time() - t2))
  t3 = time.time()
  multi_thread()
  print('多线程：%fs' % (time.time() - t3))
  ```

  结果：

  ![](/madao.github.io/database/images/articles/python/asyncio/image1.png)

  协程是和多线程的速度差不多的，但是协程是在一个线程里实现的，没有线程切换的开销，所以线程越多，协程的性能优势就越明显。

### 三. 关于生成器的一些总结

之前在生成器和迭代器中总结过这方面的一些知识，但是看到协程的使用，好多大神都会用使用了 yield 的函数作为例子讲解，但是我并不是很理解。所以在这里继续总结一些之前没有总结到的知识点：

- 生成器的执行顺序

  我一直以为使用了 yield 的函数（也就是生成器），是执行到了 yield 返回 yield 后面的跟着的值，然后继续执行下面的代码，然后第二次执行的时候再从头开始，其实顺序不是这样的，是第一次执行到 yield 处，然后返回 yield 后面的跟着的值，函数就暂停了，等下一次再执行的时候就从上一次暂停的 yield 处继续执行，例子：

  ```
  def my_generator(max):
  n = 0
  while n <= max:
      print('yield前')
      yield n
      print('yield后')
      n += 1


  g = my_generator(10)
  print(next(g))
  print('=' * 10)
  print(next(g))
  ```

  结果：

  ![](/madao.github.io/database/images/articles/python/asyncio/image2.png)

  可以看到第一次使用 next 获取迭代器 g 的值得时候，是到了 yield 前然后返回，就停了，第二次则是从上次停止的地方继续执行下面的代码，碰到 yield 返回，函数暂停。

- next，send，close，throw

  生成器有几种状态，可以通过 getgeneratorstate 方法看到：

  ![](/madao.github.io/database/images/articles/python/asyncio/image3.png)

  ```
  GEN_CREATED # 等待执行
  GEN_RUNNING # 正在执行
  GEN_SUSPENDED # 暂停(在yield处)
  GEN_CLOSED # 执行结束
  ```

  当生成器第一次被调用的时候是没有办法拿到 yield 返回的结果的：

  ```
  from inspect import getgeneratorstate


  def my_generator(max):
      n = 0
      while n <= max:
          yield n
          n += 1


  g = my_generator(10)
  print(g)  # <generator object my_generator at 0x102804d68>
  print(getgeneratorstate(g)) # GEN_CREATED
  ```

  想要拿到 yield 返回的结果，就要用的 send 或者 next 方法，相当于激活（启动）生成器。例子：

  ```
  def my_generator(max):
      n = 0
      while n < max:
          m = yield n
          n += m


  g = my_generator(10)
  print(next(g))   # 0
  print(g.send(4)) # 4
  print(g.send(8)) # 8
  ```

  next 和 send 方法都可以拿到生成器 yield 返回的结果，它们的不同就是 send 可以带一个参数，这个参数指的是上一次 yield 语句的返回值。分析一下上面的例子就明白了：

  ```
  # 生成器
  def my_generator(max):
      n = 0
      while n < max:
          m = yield n
          n += m

  # 创建一个generator对象，并没有执行yield语句
  g = my_generator(10)

  '''
  激活生成器，并执行它，执行到yield n，返回n，注意
  m = yield n的执行顺序是从右到左，也就是yield n之后函数就暂停了
  m这时候并没有被赋值。这时候n = 0
  '''
  print(next(g))

  '''
  send 方法指定了一个上次 yield n 的返回值为4，这已经是第二次执行生成器了
  所以从上次停止的地方开始，上次是停在了给m赋值的操作，yield n返回值被指定为
  4，所以m被赋值为4，继续执行n += m这句代码，这时候n = 4, while语句的条件为True，
  所以继续循环到了 m = yield n这句代码，同样从右到左的顺序，执行 yield n 返回n的值4
  '''
  print(g.send(4))

  '''
  以此类推，上次还还是停在了给m赋值的操作，send方法的参数是8，所以上次 yield n 这句代码返回的就是8，把8赋值给m，继续执行，当再一次执行到m = yield n的时候，n就是12了。
  '''
  print(g.send(8))
  ```

  如果上面解释的还是不清楚的话，可以这样理解，就是 yield n 这句话会让生成器返回 n 的值，然后 yield n 这句话本身也会返回一个值，send 方法的参数就是指定 yield n 这句话本身的返回值。

  有时候还会看到这样的代码：

  ```
  example_generator.send(None)
  ```

  send(None)就好像第一次调用生成器的 next(example_generator)一样，都可以激活一个生成器，只不过激活的时候 send 的参数只能是 None。例子：

  ```
  def my_generator(max):
  n = 0
  while n < max:
      m = yield n
      n = m


  g = my_generator(100)
  # g.send(None) == next(g)
  print(g.send(None))  # 0
  print(g.send(4))  # 4
  print(g.send(8)) # 12
  ```

  close 方法就和它的名字一样，关闭一个生成器。当关闭之后再通过 next 或者 send 方法就会报 StopIteration。例子：

  ```
  def my_generator(max):
  n = 0
  while n < max:
      m = yield n
      n = m


  g = my_generator(100)
  print(g.send(None))
  print(g.send(4))
  g.close()
  print(g.send(8))
  ```

  ```
  # 结果
  0
  4
  Traceback (most recent call last):
      File "test.py", line 16, in <module>
          print(g.send(8))
  StopIteration
  ```

  throw 方法可以结束生成器执行，并抛出指定异常或系统定义异常

  ```
  def my_generator(max):
  try:
      n = 0
      while n < max:
          m = yield n
          n = m
  except ValueError:
      print('ValueError')


  g = my_generator(100)
  print(g.send(None))  # 0
  print(g.send(4))     # 4
  g.throw(ValueError)  # ValueError
  ```

### 四. asyncio

说了那么多，终于到了 asyncio，asyncio 是 Python 内置的一个标准库，是 Python3.4 版本之后引入的，也就是 Python 内置了对异步 IO 的支持。

asyncio 的使用：

1. async/await 关键字

   给函数前面加上 async 关键字可以定义一个协程，直接调用这个协程的函数，并不会直接执行这个函数，而是返回一个 coroutine 对象，而且还会引发一个 RuntimeWarning 的警告

   ```
   import asyncio


   async def say_hello():
       print('Hello')
       await asyncio.sleep(1)
   ```

   ```
   # 结果
   <coroutine object say_hello at 0x10e797ec8>
   test.py:8: RuntimeWarning: coroutine 'say_hello' was never awaited
       print(say_hello())
   RuntimeWarning: Enable tracemalloc to get the object allocation traceback
   ```

   await 则是挂起一个耗时的操作，也就是告诉 Python 这一步是耗时的，主线程不用等待这个操作，可以切换去执行其他协程。

   例子：

   ```
   import asyncio


   async def say_hello(i):
       print('Hello start %d ' % i)
       await asyncio.sleep(1)
       print('Hello end %d' % i)


   async def main(loop):
       tasks = [loop.create_task(say_hello(i)) for i in range(1, 3)]
       await asyncio.wait(tasks)


   loop = asyncio.get_event_loop()
   loop.run_until_complete(main(loop))
   ```

   结果：

   ```
   Hello start 1
   Hello start 2
   Hello end 1
   Hello end 2
   ```

   使用 await，主线程就不会去等待 asyncio.sleep(1)这个操作了，它会转去执行其他协程。只有协程, Task 和 Future 才可以被 await 可以成功挂起，比如
   `await time.sleep(1)`就不会被挂起

2. asyncio.sleep

   asyncio.sleep 有点像 time.sleep，但是 time.sleep 会阻塞主线程，这个不会，它会阻塞当前的任务，让 Python 去执行其他的任务，参数就阻塞的时间。

3. loop.create_task

   说 loop.create_task 之前，最好先知道这两个概念：

   > Future 对象: 包含了异步操作结果的对象。

   > Task 对象：继承自 Future，它是对 Future 和协程的进一步封装，用于事件循环。

   我这个是抄别人博客中的解释，[原博客地址](https://oee.ooo/archives/162.html)

   说实话还是不理解这个两个概念，目前的理解是，协程函数不能直接运行，会报警告（之前的例子中有），想要运行协程函数就得将它包装成 Task 对象。包装的方法有两个：

   - asyncio.ensure_future(coro_or_future, \*, loop=None)
   - loop.create_task(coroutine)

   官方推荐使用 loop.create_task，ensure_future 接受 future 对象或者协程对象，create_task 只接受协程对象。

4. asyncio.wait

   asyncio.wait 是同时运行任务的方法，和它很像的还有一个 asyncio.gather。看下它们两的区别：

   ```
   import asyncio


   async def foo(i):
       return i ** i


   async def work_1(loop):
       tasks = [asyncio.create_task(foo(i)) for i in range(4)]
       done, pending = await asyncio.wait(tasks)
       print(done, pending)
       for task in done:
           print(task.result())

   loop = asyncio.get_event_loop()
   loop.run_until_complete(work_1(loop))
   ```

   ```
   # 结果：

   {<Task finished coro=<foo() done, defined at test.py:4> result=1>, <Task finished coro=<foo() done, defined at test.py:4> result=1>, <Task finished coro=<foo() done, defined at test.py:4> result=4>, <Task finished coro=<foo() done, defined at test.py:4> result=27>} set()
   1
   1
   4
   27
   ```

   - 第一个参数是包含 coroutines 或 futures 的可迭代对象。
   - 执行是无序的。
   - 返回值是完成的任务和未完成的任务，通过 result 方法获取任务的结果

   ```
   import asyncio


   async def foo(i):
       return i ** i



   async def work_2(loop):
       tasks = [asyncio.create_task(foo(i)) for i in range(4)]
       result = await asyncio.gather(*tasks)
       print(result)

   loop = asyncio.get_event_loop()
   loop.run_until_complete(work_2(loop))
   ```

   ```
   # 结果：

   [1, 1, 4, 27]
   ```

   - 第一个参数是任意个 coroutines 或 futures。
   - 执行是有序的
   - 返回值是已完成任务的结果

5. loop = asyncio.get_event_loop

   得到当前上下文的事件循环。

   这句话一下子搞了两个对我来说很模糊的概念，就是事件循环和上下文：

   - 事件循环：

     > 在计算系统中，可以产生事件的实体叫做事件源，能处理事件的实体叫做事件处理者。此外，还有一些第三方实体叫做事件循环。它的作用是管理所有的事件，在整个程序运行过程中不断循环执行，追踪事件发生的顺序将它们放到队列中，当主线程空闲的时候，调用相应的事件处理者处理事件。

     [原文](https://python-parallel-programmning-cookbook.readthedocs.io/zh_CN/latest/chapter4/03_Event_loop_management_with_Asyncio.html)

   - 上下文：

     > 上下文是一段程序运行所需要的最小数据集合。

     [原文](http://tianfangye.com/2017/10/14/context-and-scope-bonus-scene-closure/)

   在协程中，把协程函数注册（放入）事件循环中，当事件发生的时候就会调用对应的协程函数。

6. loop.run_until_complete

   运行直到传入的 Future 对象完成。run_until_complete 方法可以接受 Future 对象也可以接受协程对象，如果传入的是协程对象，它会帮你转换成 Future 对象。

7. loop.close()

   关闭事件循环。

基本上是把例子中用到的都说了，虽然还是有点不清楚，先记下来，下一步就是应用在爬虫上，希望可以在用的时候加深理解。
