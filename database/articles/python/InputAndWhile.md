- input()

  input()方法会让程序暂停等待用户输入，当用户输入完成后，我们可以获得用户输入的文本，input()方法接受一个字符串，这个字符串会在用户输入的地方展示出来，例子：

  ```
  name = input("What's your name？")
  print('Hello!', name)
  ```

  ![](/madao.github.io/database/images/articles/python/input_and_while/image.png)

  需要注意的是，input 获取到的值，都是字符串，例子：

  ```
  age = input("How old are you: ")
  print(type(age))
  ```

  ![](/madao.github.io/database/images/articles/python/input_and_while/image1.png)

- while 循环

  while 循环和 for 循环的区别是，for 循环依赖值的数量，比如字符串的字符数量，列表的元素数量，而 while 循环，只要条件满足就可以一直循环下去。例子：

  ```
  current_number = 1
  while current_number < 10:
      print('times:', current_number)
      current_number += 1

  # times: 1
  # times: 2
  # times: 3
  # times: 4
  # times: 5
  # times: 6
  # times: 7
  # times: 8
  # times: 9
  ```

  所以使用 while 的时候一定要注意不要造成不可控的无限循环。如果把上面例子中的`current_number += 1`这句代码删除，那么就会无限循环。

  - break

  break 语句可以退出循环，包括 for 循环，例子：

  ```
  # 当用户输入的不是quit，那么就一直循环
  active = True
  message = ''
  while active:
      message = input('Say something：')
      if message == 'quit':
          break
      else:
          print(message)
  ```

  ![](/madao.github.io/database/images/articles/python/input_and_while/image2.png)

- continue

  continue 和 break 的区别是，continue 是退出这一次的循环，break 是退出整个循环，比如，只要偶数才打印数字就可以这样写：

  ```
  for value in range(10):
    if value % 2 == 0:
        print(value)
    else:
        continue
    # 0
    # 2
    # 4
    # 6
    # 8
  ```
