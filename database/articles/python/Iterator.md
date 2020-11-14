### 一. 迭代器

- 可迭代对象：可以使用 for 循环的对象叫做可迭代对象。例子：

  ```
  for i in [1, 2, 3]:
      print(i)

  test_dict = {'name': 'allen', 'age': 17}
  for key, value in test_dict.items():
      print(key, value)
  ```

- 迭代器：可以被 next()函数调用并不断返回下一个值的对象称为迭代器。

- iter：列表，字典都可以叫做可迭代对象，但是他们不是迭代器，如果要将它们变成迭代器，就要使用 iter()函数，例子：

  ```
  test_list = [1,2,3,4]

  iter_test_list = iter(test_list)

  print(next(iter_test_list))
  print(next(iter_test_list))
  print(next(iter_test_list))
  print(next(iter_test_list))
  print(next(iter_test_list))

  ```

  ![](/madao.github.io/database/images/articles/python/iterator/image.png)

  每次调用 next 函数，迭代器会返回下一个值，如果下一个值没有，那么就会报错。

### 二. 生成器

生成器是一个返回迭代器的函数，在 Python 中使用了 yield 的函数被称为生成器。

range 函数可以生成一个整数序列，但是它的 step（步长）参数，只能是整数。如果想要 step 为浮点数要怎么实现？

```
# 实现

def float_range(start, stop, step):
    x = start
    while x < stop:
        yield x
        x += step
```

![](/madao.github.io/database/images/articles/python/iterator/image.png)

注意例子中的 yield 关键字，在调用生成器运行的过程中，每次遇到 yield 时函数会暂停并保存当前所有的运行信息，返回 yield 的值, 并在下一次执行 next() 方法时从当前位置继续运行。
