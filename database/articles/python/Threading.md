- ### 区分进程和线程

  线程和进程是操作系统相关的概念，比如在电脑上可以一边听歌，一边浏览网页，当你打开一个音乐播放器的时候就会启动一个进程，打开浏览器的时候也会启动一个进程。

  ![](/caisr.github.io/database/images/articles/python/threading/image.png)

  在音乐播放器中我们在听歌的同时，发表一些评论或者搜索其他歌曲。在进程里同时执行的这些任务，叫做线程。一个进程里是可以包含多个线程的。

- ### Python 多线程编程

  Python 中多线程编程需要借助的模块是：threading

  - 单线程和多线程对比：

    - 单线程：

      ```
      import time


      # 单线程
      def sayHello(index):
          print('hello!', index)
          time.sleep(1)
          print('执行完成')


      for i in range(5):
          sayHello(i)
      ```

      结果：
      ![](/caisr.github.io/database/images/articles/python/threading/image1.png)
      例子中看单线程程序执行呢就是一个接着一个执行

    - 多线程：

      ```
      import time
      import threading


      def sayHello(index):
          print('hello!', index)
          time.sleep(1)
          print('执行完成')


      for i in range(5):
          t1 = threading.Thread(target=sayHello, args=(i,))
          t1.start()
      ```

      结果：

      ![](/caisr.github.io/database/images/articles/python/threading/image2.png)

      多线程则像是超市的收银台一样，2 号收银台不用等 1 号收银台的顾客全部交完钱了再开始工作，可以一起工作。

- ### threading.Thread
  threading.Thread 函数，创建一个线程。
  使用 help 函数可以看到它接受的参数有这些：
  ```

  Thread(group=None, target=None, name=None, args=(), kwargs=None, *, daemon=None)



  |*group* should be None; reserved for future extension when a ThreadGroup

  |class is implemented.

  |

  |*target* is the callable object to be invoked by the run()

  |method. Defaults to None, meaning nothing is called.

  |

  |*name* is the thread name. By default, a unique name is constructed of

  |the form "Thread-N" where N is a small decimal number.

  |

  |*args* is the argument tuple for the target invocation. Defaults to ().

  |

  |*kwargs* is a dictionary of keyword arguments for the target

  |invocation. Defaults to {}.

  ```
  target 就是线程启动后执行的函数
  args 和 kwargs 都是这个函数的参数，args 是元组类型，kwargs 是关键字类型参数。
  name 是给这个线程的名字，如果不传就是 Thread-1, Thread-2....
  它还有一些方法：
  - run(): 执行线程中的方法。
  - start(): 启动线程。
  - join(): 等待线程结束。
  - isAlive(): 返回线程是否活动的。
  - getName(): 返回线程名。
  - setName(): 设置线程名。
- ### threading.currentThread

  threading.currentThread 可以获得当前的线程信息，例子：

  ```
  import time
  from threading import Thread, currentThread

  def sayHello(index):
      print(currentThread().getName(), index)
      time.sleep(1)
      print('执行完成')


  for i in range(5):
      t1 = Thread(target=sayHello, kwargs={'index': 1})
      t1.start()
  ```

  结果：

  ![](/caisr.github.io/database/images/articles/python/threading/image3.png)

结合 Thread 类的 join 方法，可以看下主线程和子线程之间的顺序：

```
from threading import Thread, currentThread
import time


def do_somethong():
    print(currentThread().getName(), '开始')
    time.sleep(1)
    print(currentThread().getName(), '结束')


t1 = Thread(target=do_somethong)
t1.start()
print(currentThread().getName(), '结束')

```

结果：

![](/caisr.github.io/database/images/articles/python/threading/image4.png)

MainThread 就是主线程，可以看到是主线程先结束 Thread-1 线程再结束的，如果想要主线程最后结束，那么就可以使用 join 方法了。

```
from threading import Thread, currentThread
import time


def do_somethong():
    print(currentThread().getName(), '开始')
    time.sleep(1)
    print(currentThread().getName(), '结束')


t1 = Thread(target=do_somethong)
t1.start()
t1.join()  # 新增
print(currentThread().getName(), '结束')

```

![](/caisr.github.io/database/images/articles/python/threading/image5.png)

- ### 多个线程之间的变量

    ```
    from threading import Thread
    a = 0


    def change_num(num):
        global a
        a += 1
        a -= 1


    def run_thread(num):
        for i in range(100000):
            change_num(num)


    t1 = Thread(target=run_thread, args=(1,))
    t2 = Thread(target=run_thread, args=(2,))

    t1.start()
    t2.start()
    t1.join()
    t2.join()

    print(a)
    ```

    多次运行上面代码的结果：

    ![](/caisr.github.io/database/images/articles/python/threading/image6.png)

    a 的值应该永远是 0 的，但是有时候会变，多线程中的变量是共享的，如果同时去改一个变量，那么有可能会将该变量改乱。[原因](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/00143192823818768cd506abbc94eb5916192364506fa5d000)

    如果想要多个线程修改同一变量，那么就需要一个锁来确保该变量修改后的正确性。
    锁就是当一个线程在修改的时候，其他线程等待。这个线程修改完了，下个线程继续修改。

    实现这个锁，要用到 threading.lock()方法：

    ```

    from threading import Thread, Lock

    a = 0

    thread_lock = Lock()  # 创建一个锁





    def change_num(num):

        global a

        a += 1

        a -= 1





    def run_thread(num):

        for i in range(100000):

            try:

                thread_lock.acquire()  # 锁住

                change_num(num)

            finally:

                thread_lock.release() # 解锁





    t1 = Thread(target=run_thread, args=(1,))

    t2 = Thread(target=run_thread, args=(2,))



    t1.start()

    t2.start()

    t1.join()

    t2.join()



    print(a)

    ```

- ### threading.local

  线程之间如果使用局部变量，那么就可以用 threading.local 这个方法。例子：

  ```
  from threading import Thread, currentThread, local


  data = local()


  def do_somethong(name):
      data.name = name
      print(currentThread().getName(), data.name)


  t1 = Thread(target=do_somethong, args=('jack',))
  t2 = Thread(target=do_somethong, args=('alex',))
  t1.start()
  t2.start()
  t1.join()
  t2.join()

  ```

  结果：

  ![](/caisr.github.io/database/images/articles/python/threading/image7.png)

  用 threading.local 的好处是，当线程中函数比较复杂的时候，也就是说不仅仅只是执行一个函数，而是多个函数，比如：

  ```
  from threading import Thread
  import time


  def sayHelle(name):
      print('hello!%s' % name)

  def sayBye(name):
      print('goodbye!%s' % name)

  def run(name):
      sayHelle(name)
      time.sleep(1)
      sayBye(name)

  t1 = Thread(target=run, args=('jack',))
  t1 = Thread(target=run, args=('jack',))

  t1.start()
  t2.start()
  t1.join()
  t2.join()
  ```

  这个 name 参数就得一个一个的传递下去，这样很不方便，threading.lock()方法就像是全局的一个字典，每一个线程从里面获取 name 的时候都是各自线程自己存进去的 name，相互不会干扰。
