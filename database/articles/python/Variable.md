在 Python 中，如果希望程序记住某个值，以便以后使用，所要做的就是给这个值起一个名字，使用的时候只要使用这个名字，就可以得到之前保存的值。这个名字就叫做变量。

例子：

```

message = 'I hate group-building'

print(message)

```

上面代码中，message 就是一个变量，它的值是一个字符串。

### 1. 变量的命名和使用

- 变量名只能包含字母，数组和下划线。变量可以用字面或下划线开头，但不能用数字开头。例子：
  ```

  # 正确

  message_1 = 'a'

  # 错误

  1_message = 'b'

  ```
- 变量名不能包含空格，可以使用下划线来分割单词。例子：
  ```

  # 正确

  city_name = '广州'

  # 错误

  city name = '广州'

  ```
- 不要将 Python 关键字和内置函数名用作变量名，关键字和内置函数有：

  关键字

  ```
  help('keywords')
  ```

  可用通过内置函数，获得所有的关键字

  ![](/caisr.github.io/database/images/articles/python/variable/image.png)

  内置函数

  参考官方文档[Built-in Functions](https://docs.python.org/3.7/library/functions.html?highlight=built)

  ![](/caisr.github.io/database/images/articles/python/variable/image1.png)

- 变量名应该简短又具有描述性（起名真的很难。。），例子：

  ```

  # good

  name

  # bad

  n



  # good

  name_length

  # bad

  length_of_persons_name

  ```

- 慎用大写字母 I 和大写字母 O，容易看成数字 1 和 0。
- 变量名是区分大小写的。

在程序中可以随时修改变量的值，但是 Python 只会记录变量的最新值，例子：

```

message = 'Hate to work'

print(message)



message = 'Like to go to work'

print(message)

```

![](/caisr.github.io/database/images/articles/python/variable/image2.png)
