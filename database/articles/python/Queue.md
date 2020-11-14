- ### 生产者和消费者模型

  在多线程开发的过程中有可能遇到这样一种情形：

  有一部分线程负责生产一些数据，有一部分负责处理一些数据，把制造数据的那一部分线程叫做生产者，处理数据的那一部分线程叫做消费者，如果生产者生产的数据速度大于消费者处理数据的速度，那么生产者就必须要等待消费者处理完成才能继续生产，同样消费者处理数据的速度大于生产者，那消费者就要等待生产者，为了解决这个问题，于是引入了生产者和消费者模型。

  生产者和消费者模型：

  ```
  生产者 ---->   缓冲区（仓库） ----> 消费者
  ```

  生产者和消费者不直接通讯，生产者生产的数据丢给缓冲区，消费者拿数据则是向缓冲区拿。

  可以将生产者和消费者模型想象成寄信：

  1. 发件人就是生产者，写信
  2. 邮箱就是缓冲区，将信放入邮箱。
  3. 邮递员从邮箱中拿出信，进行后续的处理就是消费者

  要实现这样的模型，需要用到队列，队列就是生产者和消费者之间的缓冲区。

- ### queues 模块

  > 队列，又称为伫列（queue），是先进先出（FIFO, First-In-First-Out）的线性表。在具体应用中通常用链表或者数组来实现。队列只允许在后端（称为 rear）进行插入操作，在前端（称为 front）进行删除操作。 - 维基百科

  queues 模块实现多生产者，多消费者队列。

  - queue.Queue(maxsize=0)

    FIFO（先进先出） 队列的构造方法。 maxsize 是一个整数，它设置了可以放入队列中的项目数量的上限。一旦达到此大小，插入就会阻塞，直到队列项被消耗。如果 maxsize 小于或等于零，队列大小是无限的。

  - queue.LifoQueue(maxsize=0)

    LIFO（后进先出） 队列的构造方法。 maxsize 是一个整数，它设置了可以放入队列中的项目数量的上限。一旦达到此大小，插入就会阻塞，直到队列项被消耗。如果 maxsize 小于或等于零，队列大小是无限的。

  - queue.PriorityQueue(maxsize=0)

    优先级队列的构造函数。 maxsize 是一个整数，它设置了可以放入队列中的项目数量的上限。一旦达到此大小，插入就会阻塞，直到队列项被消耗。如果 maxsize 小于或等于零，队列大小是无限的。

    存放数据的格式: Queue.put((priority_number,data))，priority_number 越小，优先级越高，data 代表存入的值

  通过例子看这三种队列的不同之处：

  ```
  import queue

  # 先进先出
  q_FIFO = queue.Queue(3)
  q_FIFO.put(1) # 存第一个数据
  q_FIFO.put(2) # 存第二个数据
  q_FIFO.put(3) # 存第二个数据

  print('取第一个数据', q_FIFO.get())
  print('取第二个数据', q_FIFO.get())
  print('取第三个数据', q_FIFO.get())
  ```

  结果：

  ![](/caisr.github.io/database/images/articles/python/queue/image.png)

  拿出的数据的顺序和存放进去数据的顺序是一样的

  ```
  # 后进先出
  q_LIFO = queue.LifoQueue(3)

  q_LIFO.put(1)  # 存第一个数据
  q_LIFO.put(2)  # 存第二个数据
  q_LIFO.put(3)  # 存第二个数据

  print('取第一个数据', q_LIFO.get())
  print('取第二个数据', q_LIFO.get())
  print('取第三个数据', q_LIFO.get())
  ```

  结果：
  ![](/caisr.github.io/database/images/articles/python/queue/image1.png)

  越后放进去的数据越先取出来

  ```
  # 优先级队列
  q_Priority = queue.PriorityQueue(3)

  q_Priority.put((10, '我的优先级是10'))  # 存第一个数据
  q_Priority.put((-1, '我的优先级是-1'))  # 存第二个数据
  q_Priority.put((1, '我的优先级是10'))  # 存第三个数据

  print('取第一个数据', q_Priority.get())
  print('取第二个数据', q_Priority.get())
  print('取第三个数据', q_Priority.get())
  ```

  结果：

  ![](/caisr.github.io/database/images/articles/python/queue/image2.png)

  它根据传入的(priority_number,data)格式的数据中的 priority_number 决定优先级的大小，priority_number 越小优先级越高。

- ### Queue，LifoQueue，PriorityQueue 队列提供的方法

  - Queue.qsize()

    返回当前队列的长度，但是结果并不可靠，原因是多线程情况下，有可能在 Queue.qsize()拿到的结果是 1，但是其他线程有可能在当前线程获取数据之前已经拿走数据了，这就会导致当前线程拿数据的时候队列的长度变成 0，所以 Queue.qsize 的结果是不可靠的。

    例子：

    ```
    import queue
    from threading import Thread
    import time

    # 创建队列
    q = queue.Queue(3)

    # 消费者1
    def print_something():
        q_size = q.qsize()
        if q_size:
            print('队列长度为：%d' % q_size)
            time.sleep(1)  # 等一秒再处理数据
            print('消费者1', q.get())
            q.task_done()  # task_done方法等下会说，可理解为通知队列当前获取到的数据已处理完毕

    # 生产者1
    def put_something():
        q.put(1)
        print('生产者1')


    # 消费者2
    def early_treatment():
        print('消费者2', q.get())
        q.task_done()


    t1 = Thread(target=put_something)
    t2 = Thread(target=print_something)
    t3 = Thread(target=early_treatment)

    t1.start()
    t2.start()
    t3.start()

    t1.join()
    t2.join()
    t3.join()
    ```

    结果：

    ![](/caisr.github.io/database/images/articles/python/queue/image3.png)

    因为消费者 1 等了一秒才开始处理数据，另一个线程的消费者 2 则是立即处理数据，所以数据当消费者 1 处理数据的时候，队列已经空了，没有数据了，消费者 1 就只能一直等待。

  - Queue.empty()

    如果队列为空，返回 True，否则返回 False。它和 qsize 一样，结果也是不可靠的。

  - Queue.full()

    如果队列已满，返回 True，否则返回 False。它和 qsize 一样，结果也是不可靠的。

  - Queue.put(item, block=True, timeout=None)

    将 item 放入队列。

    - item：放入队列的数据
    - block：是否阻塞
    - timeout：阻塞等待时间，必须是非负数或者 None

    例子：

    - block 为 True 且 timeout 为 None，如果队列满了，那么会一直等待，直到队列有空位再进行放入。

      ```
      import queue
      import datetime

      # 创建队列
      q = queue.Queue(3)

      # block为True且timeout为None
      try:
          for i in range(4):
              q.put(i)
              old_time = datetime.datetime.now()
      except queue.Full:
          new_time = datetime.datetime.now()
          print('阻塞等待时间：%s秒' % (new_time - old_time).seconds)
      ```

    - block 为 True 且 timeout 为 0，如果队列满了，立即抛出 queue.Full 异常。

      ```
      import queue
      import datetime

      # 创建队列
      q = queue.Queue(3)

      # block为True且timeout为0
      try:
          for i in range(4):
              q.put(i, timeout=0)
              old_time = datetime.datetime.now()
      except queue.Full:
          new_time = datetime.datetime.now()
          print('阻塞等待时间：%s秒' % (new_time - old_time).seconds)
      ```

      结果：

      ![](/caisr.github.io/database/images/articles/python/queue/image4.png)

    - block 为 True 且 timeout 为正数，如果队列满了，会等待 timeout 中传入的秒数，如果超过了 timeout 中传入的秒数，抛出 queue.Full 异常。

      ```
      import queue
      import datetime

      # 创建队列
      q = queue.Queue(3)

      # block为True且timeout为正数
      try:
          for i in range(4):
              q.put(i, timeout=3)
              old_time = datetime.datetime.now()
      except queue.Full:
          new_time = datetime.datetime.now()
          print('阻塞等待时间：%s秒' % (new_time - old_time).seconds)
      ```

      结果：

      ![](/caisr.github.io/database/images/articles/python/queue/image5.png)

    - block 为 False，队列满了会立即抛出 queue.Full 异常。

  - Queue.put_nowait(item)

    相当于 put(item, block=False)。

  - Queue.get(block=True, timeout=None)

    从队列中删除并返回项目。block 和 timeout 参数的含义和 put 方法相同，不过执行的操作是拿出不是放入。当队列为空时，继续 get 会抛出 queue.Empty 异常。

  - Queue.get_nowait()

    相当于 get(False)。

  - Queue.task_done()

    通知队列当前使用 get 获取的任务已完成。

  - Queue.join()

    阻塞，直到队列中的所有任务都被获取和处理。

- ### 简单的生产者消费者模型

  - 先生产后消费

    ```
    import threading
    import queue
    import random

    def producer():
        """
        模拟生产者
        """
        for i in range(10):
            q.put("%s元" % random.randint(1, 100))

        print("等待所有的钱被取走...")
        q.join()
        print("所有的钱被取完了...")


    def consumer(n):
        """
        模拟消费者
        """
        while q.qsize() > 0:
            print("%s 取到" % n, q.get())
            q.task_done()


    q = queue.Queue()

    p1 = threading.Thread(target=producer)
    p1.start()

    c1 = consumer("allen")
    ```

  - 边生产边消费

    ```
    import threading
    import queue
    import random
    import time


    def producer():
        """
        模拟生产者
        """
        while True:
            time.sleep(random.randint(1, 5))
            data = random.randint(1, 100)
            q.put("%s元" % data)
            print('存入：%s元' % data)


    def consumer(n):
        """
        模拟消费者
        """
        while True:
            time.sleep(random.randint(1, 5))
            data = q.get()
            print("%s 取到" % n, data)
            q.task_done()


    q = queue.Queue()
    names = ['allen', 'jack']
    p1 = threading.Thread(target=producer)
    p1.start()
    for name in names:
        c = threading.Thread(target=consumer, args=(name,))
        c.start()


    ```
