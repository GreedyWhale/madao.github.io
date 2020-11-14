### 1. 字符串

在 Python 中，用引号包起来的都是字符串。

- 引号

  字符串的引号必须开头和结尾一样，也就是说双引号（"）开头的字符串，必须以双引号结尾，单引号（'）开头的字符串，必须以双单号结尾，双引号中可以包含单引号，同样单引号中可以包含双引号。例子：

  ```

  sentence = 'Love is a touch and yet not a touch."——Jerome David Salinger"'

  print(sentence)



  sentence = "Love is a touch and yet not a touch.'——Jerome David Salinger'"

  print(sentence)

  ```

  ![](/madao.github.io/database/images/articles/python/string_and_number/image.png)

- 拼接

  ```
  print('cat' + 'dog')
  ```

  ![](/madao.github.io/database/images/articles/python/string_and_number/image1.png)

- 重复

  在 Python 中可以用\*来重复。例子：

  ```
  print('cat' * 20)
  ```

  ![](/madao.github.io/database/images/articles/python/string_and_number/image2.png)

  但是不能这样：

  ```
  print('cat' + 5)
  ```

  ![](/madao.github.io/database/images/articles/python/string_and_number/image3.png)

- 字符串的大小写

  Python 中提供了 title()，lower()，upper()这几内置函数来改变字符串的大小写，例子:

  ```

  name = 'allen walker'

  print(name.title())

  print(name.lower())

  print(name.upper())

  ```

  ![](/madao.github.io/database/images/articles/python/string_and_number/image4.png)

  title 会让字符串中的单词首字母大写，lower 就是全部小写，upper 为全部大写

- 换行符和制表符

  Python 中可以使用\n 和\t 代表换行符和制表符，例子：

  ```

  print('hobby:\ngakki\nanime\nsleep')

  print('hobby:\n\tgakki\n\tanime\n\tsleep')

  ```

  ![](/madao.github.io/database/images/articles/python/string_and_number/image5.png)

- 删除空白

  Python 中提供了 strip()，lstrip()，rstrip()来删除字符串中的空白：

- strip 可以将字符串两端的空白删除。
- lstrip 可以字符串左端的空白删除。
- rstrip 可以字符串右端的空白删除。
  ```

  language = '  Python      '

  language1 = '  JavaScript      '

  language2 = '  Java      '

  print(language.strip())

  print(language1.lstrip())

  print(language2.rstrip())

  ```
  ![](/madao.github.io/database/images/articles/python/string_and_number/image6.png)
- 长字符串

  Python 中过长的字符串可以用"""或者'''来包裹，例子：

  ```

  long_string = """Grasshoppers are insects in the suborder Caelifera, probably

  the oldest living group of chewing herbivorous insects, dating back to the

  early Triassic around 250 million years ago. They are typically

  ground-dwelling insects with powerful hind legs which enable the"""



  long_string1 = '''\n\nGrasshoppers are insects in the suborder Caelifera, probably

  the oldest living group of chewing herbivorous insects, dating back to the

  early Triassic around 250 million years ago. They are typically

  ground-dwelling insects with powerful hind legs which enable the'''



  print(long_string, long_string1)

  ```

  ![](/madao.github.io/database/images/articles/python/string_and_number/image7.png)

### 2. 数字

Python 中数字类型分为整数和浮点数，浮点数就是带小数点的数字。
可以使用 加（+）、减（-）、乘（\*）、除（/）进行运算。除了这 4 种，Python 种还可以进行指数（\*\*）和 取余（%）运算。例子：

![](/madao.github.io/database/images/articles/python/string_and_number/image8.png)

其中 `3**5`就是 3 的 5 次幂。

需要注意的一点，Python3 中整除需要使用 `//` 操作符。例子：

![](/madao.github.io/database/images/articles/python/string_and_number/image9.png)

当一个数特别大的时候 Python 会采用 E 记法。例子：

![](/madao.github.io/database/images/articles/python/string_and_number/image10.png)

E 记法传送门：[科学记数法](https://zh.wikipedia.org/zh-hans/%E7%A7%91%E5%AD%A6%E8%AE%B0%E6%95%B0%E6%B3%95)

在 Python 中 0.1 + 0.2 也是不等于 0.3 的，例子：

![](/madao.github.io/database/images/articles/python/string_and_number/image11.png)

- 自增和自减

  自增的操作符是 `+=`，自减的操作符是`-=`，例子：

  ```

  number = 10

  number += 1

  print(number) # 11

  number -= 1

  print(number) # 10

  ```

### 3. 类型转换

- float()，从一个字符串或整数中创建一个新的浮点数
- int()，从字符串或浮点数中创建一个新的整数
- str()，从一个数（可以是任何类型）中创建一个新的字符串

  例子：

  ![](/madao.github.io/database/images/articles/python/string_and_number/image12.png)

- type()，type 方法可以检测一个值的类型，例子：

  ![](/madao.github.io/database/images/articles/python/string_and_number/image13.png)

### 4. Python 之禅

在 python 的环境中可以用`import this`这句代码获取“Python 之禅”。例子：

![](/madao.github.io/database/images/articles/python/string_and_number/image14.png)
