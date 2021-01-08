Python 的闭包和 JavaScript 中的闭包很像，形式都是一个函数的返回值为另一个函数，返回出来的这个函数引用了它外层函数的变量。例子：

```

def foo(a):

    def bar(b):

        return a + b

    return bar

add_one = foo(1)

print(add_one(1)) # 2

print(add_one(2)) # 3

print(add_one(3)) # 4

```

### 闭包的作用

- 让函数内部的变量在执行完毕后不被回收

  例子：

  ```

  # 计数器，每调用一次就加一

  def counter():

      counts = 0

      def add_one():

          nonlocal counts

          counts += 1

          return counts

      return add_one

  counter_1 = counter()

  print(counter_1()) # 1

  print(counter_1()) # 2

  print(counter_1()) # 3

  print(counter_1()) # 4

  ```

- 在某些情况下更为简洁

  例子:

  ```

  # 做一个运算 a * x + b，x可变，a,b固定

  def line(a, b):

      return lambda x: a * x + b

  line_1 = line(10, 20)

  line_2 = line(100, 200)

  print(line_1(5)) # 70

  print(line_1(6)) # 80

  print(line_1(7)) # 90

  print(line_2(5)) # 700

  print(line_2(6)) # 800

  print(line_2(7)) # 900

  ```

  从例子中可以看出，可以给定 a,b 的值，然后每次修改 x 的时候，只传入 x 就可以了。这样是不是更简洁，每个人的看法可能会不同。
