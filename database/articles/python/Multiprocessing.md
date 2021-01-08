- ### 什么是 GIL

  > 全局解释器锁（英语：Global Interpreter Lock，缩写 GIL），是计算机程序设计语言解释器用于同步线程的一种机制，它使得任何时刻仅有一个线程在执行。即便在多核心处理器上，使用 GIL 的解释器也只允许同一时间执行一个线程。常见的使用 GIL 的解释器有 CPython 与 Ruby MRI。 -- 维基百科

  一般用的 Python 的解释器就是 CPython（官网下载的版本），结合上面对 GIL 的解释，得出两个结论：

  1. Python 中的多线程并不是同时有多个线程并行执行，而是不断的切换线程达到多线程并行的效果，同一时间只能执行一个线程。

  2. Python 的多线程程序并不能利用多核 CPU 的优势，无论有多少个线程，只能运行在一个核上。

- ### 测试 GIL

  ```
  import threading
  import time
  from queue import Queue
  def normal(num_list):
      total = sum(num_list)
      print(total)
  def mutilThread_sum(l, q):
      q.put(sum(l))
  def mutilThread(num_list):
      q = Queue()
      threads = []
      total = 0
      for idx in range(4):
          thread = threading.Thread(target=mutilThread_sum, args=(num_list[:], q))
          thread.start()
          threads.append(thread)
      [thread.join() for thread in threads]
      for thread in threads:
          total += q.get()
      print(total)
  num_list = list(range(1000000))

  normal_start = time.time()
  normal(num_list*4)
  print('normal:', time.time() - normal_start)

  mutilThread_start = time.time()
  mutilThread(num_list)
  print('mutilThread:', time.time() - mutilThread_start)
  ```

  结果：

  ![](/madao.github.io/database/images/articles/python/multiprocessing/image.png)

  单线程计算的是多线程中每个线程计算的 4 倍，理论上多线程应该比单线程快 3~4 倍左右，但是从结果看来 mutilThread 并没有比 normal 快那么多，而且如果多运行几次你还会发现多线程有时候还会比单线程慢。这就是因为 GIL 的存在导致的。

- ### 多进程

  虽然无法使用多线程利用多核 CPU 的优势，但是可以使用多进程达到这一目的。

  - multiprocessing

    Python 中多进程需要借助 multiprocessing 模块实现。这个模块和 Threading 模块很像。

    ```
    from multiprocessing import Process
    def add_one(a):
        print(a + 1)
    p1 = Process(target=add_one, args=(1,)) # 创建一个进程
    p1.start()  # 启动进程
    p1.join()   # 等待进程结束
    ```

  - 进程中函数的输出

    观察上面例子中的函数，如果把`print(a + 1)`换为`return a + 1`。那么怎样才能拿到这个函数的返回值？用队列。

    ```
    from multiprocessing import Process, Queue
    def add_one(q, a):
        q.put(a + 1)
    q = Queue()
    p1 = Process(target=add_one, args=(q, 1))
    p2 = Process(target=add_one, args=(q, 10))
    p1.start()
    p2.start()
    p1.join()
    p2.join()

    a = q.get()
    b = q.get()

    print(a + b)  # 13
    ```

- ### 简单的多进程，多线程效率对比

  ```
  from multiprocessing import Process, Queue
  from threading import Thread
  import time
  def calculate(q):
      res = 0
      for i in range(10000000):
          res += i ** 2
      q.put(res)
  # 多进程
  def multi_process():
      q = Queue()
      p1 = Process(target=calculate, args=(q,))
      p2 = Process(target=calculate, args=(q,))
      p1.start()
      p2.start()
      p1.join()
      p2.join()

      res = q.get() + q.get()
      print('multi_process:', res)
  # 多线程
  def multi_thread():
      q = Queue()
      p1 = Thread(target=calculate, args=(q,))
      p2 = Thread(target=calculate, args=(q,))
      p1.start()
      p2.start()
      p1.join()
      p2.join()

      res = q.get() + q.get()
      print('multi_thread:', res)
  # 单线程
  def normal():
      res = 0
      for i in range(2):
          for j in range(10000000):
              res += j ** 2
      print('normal:', res)
  t1 = time.time()
  normal()
  t2 = time.time()
  print('normal time:', t2 - t1)
  multi_process()
  t3 = time.time()
  print('multi_process time:', t3 - t2)
  multi_thread()
  t4 = time.time()
  print('multi_thread time:', t4 - t3)

  ```

  结果：

  ![](/madao.github.io/database/images/articles/python/multiprocessing/image1.png)

  对比一下，多进程效率是最高的，当然了这个例子是属于计算密集型的例子，在 io 密集型的程序上，多线程的优势还是挺大的。

- ### 进程池

  如果需要大量的创建进程，那么就可以使用进程池这个功能：Pool

  ```
  from multiprocessing import Pool
  from os import getpid
  import time
  import random
  def square(num):
      print('进程id：%s启动' % getpid())
      time.sleep(random.random() * 3)
      print('进程id：%s执行完毕' % getpid())
      return num ** 2
  p = Pool()
  results = []
  for i in range(10):
      res = p.apply_async(square, args=(i,))
      results.append(res)

  p.close()
  p.join()

  for result in results:
      print(result.get())

  ```

  结果：

  ![](/madao.github.io/database/images/articles/python/multiprocessing/image2.png)

  解释：

  1. pool：用于创建一个进程池对象，可以传入同时跑的进程的数量，默认的进程数是 cpu 的核数。比如：

     ```
     p = Pool(processes=10)
     ```

     这样就可以让 10 个进程同时跑，仔细观察上面例子的结果，前 8 个进程是立即就启动了，因为没有给 pool 方法传入进程数量，默认就是 cpu 核的数量。我这里是 8，所以同时只能跑 8 个进程，当这八个进程中的某一个进程的任务执行完成后才会启动新的进程。

  2. apply_async：启动进程，执行参数中传入的函数，返回的结果使用 get 方法获取，注意 get 是会阻塞的，也就是说如果写成下面这样的话，就变成了一个进程执行完成再启动下一个进程，和单进程没什么两样了：
     ```
     for i in range(10):
         res = p.apply_async(square, args=(i,))
         res.get()
     ```
  3. close：就是关闭 pool，不再接受新的进程。
  4. join：等待所有的子进程执行完毕，在调用 join 之前要先调用 close 或者 terminate 方法，terminate 方法是结束工作的进程，不管任务是否执行完成。

  如果不喜欢 apply_async 这种，还可以使用 map，例子：

  ```
  from multiprocessing import Pool
  from os import getpid
  import time
  import random
  def square(num):
      print('进程id：%s启动' % getpid())
      time.sleep(random.random() * 3)
      print('进程id：%s执行完毕' % getpid())
      return num ** 2
  p = Pool()
  res = p.map(square, range(10))

  p.close()
  p.join()

  print(res)  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
  ```
