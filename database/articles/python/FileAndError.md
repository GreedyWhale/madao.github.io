### 1. 文件读写

- 读文件

  Python 中读取一个文件使用的方法是 open()。

  [open 方法的文档](https://docs.python.org/zh-cn/3.7/library/functions.html#open)

  尝试打开一个文件：

  1.  创建一个 test.txt 文件
  2.  写入一些内容
      ```
      3.1415926535
        8979323846
        2643383279
        5028841971
      ```
  3.  新建 file_reader.py 文件，写入一下代码：
      ```
      with open('test.txt') as file_object:
          contents = file_object.read()
          print(contents)
      ```
  4.  在终端执行`python3 file_reader.py`

      结果：

      ![](/madao.github.io/database/images/articles/python/file_and_error/image.png)

  5.  如果读取的文件是二进制文件，比如图片，视频则需要用`rb`模式：
      ```
      with open('test.png', 'rb') as file_object:
      ```

  解释：

  1. with 关键字，可以让 Python 在不需要访问文件后将其关闭。这样我们就不用手动的调用`close()`方法去关闭文件了。

  2. open()方法中如果只传入文件的名字，那么 Python 会在当期目录下进行查找。

  3. read()方法会一次性读取文件全部内容，对待特别大的文件，可以使用`read(size)`，size 参数表示从文件中读取的字节数。

  4. open 方法的 mode 参数默认是 r，也就是说如果不传 mode 参数那么文件会以文本模式打开并读取。

  5. readlines 方法可以读取文件全部内容，并按行返回一个列表，注意换行符。

  6. readline 只读取一行

  ```
  with open('test.txt') as file_object:
      contentList = file_object.readlines()
      print(contentList)  # ['3.1415926535\n', '  8979323846\n', '  2643383279\n', '  5028841971']
  ```

  几种常见的 open 函数 mode 参数用法：

  | 字符 | 含义                                                                                         |
  | :--: | :------------------------------------------------------------------------------------------- |
  |  r   | 只读，指针在文件开头                                                                         |
  |  rb  | 二进制只读，指针在文件开头                                                                   |
  |  r+  | 读写，指针位置根据读写的顺序决定                                                             |
  | rb+  | 二进制读写，指针位置根据读写的顺序决定                                                       |
  |  w   | 只写，如果文件不存在则创建文件，存在则覆盖文件                                               |
  |  wb  | 二进制只写，如果文件不存在则创建文件，存在则覆盖文件                                         |
  |  w+  | 读写，如果文件不存在则创建文件，存在则覆盖文件                                               |
  | wb+  | 二进制读写，如果文件不存在则创建文件，存在则覆盖文件                                         |
  |  a   | 追加，如果文件不存在则创建文件，存在则在文件的末尾添加新内容，指针在文件末尾                 |
  |  ab  | 二进制追加，如果文件不存在则创建文件，存在则在文件的末尾添加新内容，指针在文件末尾           |
  |  a+  | 可以读可以追加，如果文件不存在则创建文件，存在则在文件的末尾添加新内容，指针在文件末尾       |
  | ab+  | 二进制可以读可以追加，如果文件不存在则创建文件，存在则在文件的末尾添加新内容，指针在文件末尾 |

- 写文件

  写文件分为两种，一种是直接覆盖原有内容，一种是在原有内容基础上添加新内容。它们对应的两种模式是

  - w

    ```
    with open('test.txt', 'w') as file_object:
       file_object.write('测试')
    ```

    test.txt 文件内容就被覆盖了

    ![](/madao.github.io/database/images/articles/python/file_and_error/image1.png)

  - a

    ```
    with open('test.txt', 'a') as file_object:
       file_object.write('随便写点什么')
    ```

    新添加的内容就会放在 test.txt 原有内容的后面：

    ![](/madao.github.io/database/images/articles/python/file_and_error/image2.png)

  既读文件又写文件 `r+`

  这里有点混乱，它和读写顺序有关还有指针的位置有关：

  1. 先读后写

     ```

     # test.txt

     test



     # file_reader.py

     with open('test.txt', 'r+') as file_object:

       contents = file_object.read()

       print(contents)

       file_object.write('Say something\n')

     ```

     运行结果：

     ![](/madao.github.io/database/images/articles/python/file_and_error/image3.png)

     test.txt 文件：

     ![](/madao.github.io/database/images/articles/python/file_and_error/image4.png)

     先读后写，写入的内容会放在原有内容的末尾，读到的内容为文件原始内容。

  2. 先写后读

     ```

     # test.txt

     test



     # file_reader.py

     with open('test.txt', 'r+') as file_object:

       file_object.write('Say something\n')

       contents = file_object.read()

       print(contents)

     ```

     运行结果：

     ![](/madao.github.io/database/images/articles/python/file_and_error/image5.png)

     test.txt 文件：

     ![](/madao.github.io/database/images/articles/python/file_and_error/image6.png)

     可以看到之前的这一行被覆盖了。而且打印出读取到的文件内容是空。

     这就牵扯到了指针问题，也就是输入的时候的闪动的光标，先写后读会覆盖原先位置上的内容，然后指针会跑到最后，读文件的时候从指针位置开始读，那么指针后面没有内容了所以是空，如果把 test.txt 文件写成这样，就会出现这样的结果：

     ```

     # test.txt

     aaaaaaaaaaaa

     bbbbbbbbbbbb

     cccccccccccc

     dddddddddddd

     eeeeeeeeeeee





     # file_reader.py

     with open('test.txt', 'r+') as file_object:

       file_object.write('xxx')

       contents = file_object.read()

       print(contents)

     ```

     结果：

     ![](/madao.github.io/database/images/articles/python/file_and_error/image7.png)

     test.txt 文件

     ![](/madao.github.io/database/images/articles/python/file_and_error/image8.png)

     看到没有，打印出来的结果中被 a 的那一行少 3 个，正是被 write 中的三个 xxx 覆盖了。打印出来的内容也是从覆盖完后的指针位置一直到结尾

     如果想要获取完整内容，那么就要用到 seek()方法，将指针移到开始的位置。

     ```

     with open('test.txt', 'r+') as file_object:

       file_object.write('xxx')

       file_object.seek(0)  # 新增

       contents = file_object.read()

       print(contents)

     ```

     结果

     ![](/madao.github.io/database/images/articles/python/file_and_error/image9.png)

  - writelines

    用 write 写入内容的时候，只能写入字符串，如果我们需要写入列表，那么就要用到 writelines 方法：

    ```
    with open('./a.txt', 'w', encoding='utf-8') as file:
        file.writelines(['1', '2', '3', '4'])
    ```

    结果：

    ![](/madao.github.io/database/images/articles/python/file_and_error/image10.png)

### 2. 异常

即使语句或表达式在语法上是正确的，但在尝试执行时，它仍可能会引发错误。 在执行时检测到的错误被称为*异常*

在 Python 中使用 try-except 代码块处理异常。

异常的类型可以参考(错误和异常)[https://docs.python.org/zh-cn/3.7/tutorial/errors.html]

```

# 只做除法的计算器



print('Enter q to quit.')



while True:

   firsr_number = input('\nFirst number: ')

   if(firsr_number) == 'q':

       break

   second_number = input('\nSecondt number: ')

   if(second_number) == 'q':

       break

   answer = int(firsr_number) / int(second_number)

   print(answer)

```

结果：

![](/madao.github.io/database/images/articles/python/file_and_error/image11.png)

正常是没问题的，但是一旦用户输入第二个数字为 0 的时候，就会异常：

![](/madao.github.io/database/images/articles/python/file_and_error/image12.png)

这时候我们就可以使用 try-except 代码块处理：

```

print('Enter q to quit.')



while True:

   firsr_number = input('\nFirst number: ')

   if(firsr_number) == 'q':

       break

   second_number = input('\nSecondt number: ')

   if(second_number) == 'q':

       break

   try:

       answer = int(firsr_number) / int(second_number)

   except ZeroDivisionError:

       print("You can't divide by zore!")

   else:

       print(answer)

```

结果：

![](/madao.github.io/database/images/articles/python/file_and_error/image13.png)

程序没有将 traceback 展示给用户，而是一句友好的提示。

还是用了 else 代码块，一般都是将有可能引发异常的代码放在 try-except 代码块中，依赖于 try 代码块成功执行的代码放在 else 代码块中。

- 抛出错误

通过 raise 语句可以抛出一个错误

```
age = 0
if age < 1:
   raise Exception('Invalid age', age)
```

![](/madao.github.io/database/images/articles/python/file_and_error/image14.png)

- pass 语句

pass 语句什么也不做，比如有一个异常，我捕获到了但是我什么也不想做，那么就可以使用 pass

```
try:
   with open('password.txt') as file_object:
       contents = file_object.read()
       print(contents)
except FileNotFoundError:
   pass
```

这样就算找不到 password.txt 文件，那么也不会报错。
