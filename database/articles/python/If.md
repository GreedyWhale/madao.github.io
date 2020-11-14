### 1. 条件测试

每条 if 语句的核心都是一个值为 True 或 False 的表达式，这种表达式被称为条件测试。Python 会根据条件测试的结果为 Ture 还是 False 来决是否执行 if 语句后面的代码。

- 判断相等
  Python 中判断相等使用两个等号（==），例子：
  ```
  car = 'BMW'
  car == 'bmw' # Fales
  car == 'BMW' # true
  ```
  Python 中判断相等是区分大小写的。
- 判断不相等
  Python 中判断不相等使用一个感叹号和一个等号（!=），例子：
  ```
  car = 'BMW'
  car != 'bmw' # true
  car != 'BMW' # false
  ```
- 数字的比较
  数字的比较除了可以用 == 和 != 还可以使用大于号（>），小于号（<）,大于等于（>=），小于等于（<=）。例子：
  ```
  print(1 < 2)  # True
  print(2 > 1)  # Ture
  print(1 <= 2) # True
  print(2 >= 1) # True
  ```
- 多个条件判断
  - and

    当需要多个条件**同时**为 True 的判断，可以用关键字 and 将测试条件合并，例子：

    ```
    print((1 <= 2) and (2 >= 1)) # True
    ```

    and 是所有的条件都为 True 的时候，表达式的结果才为 True。

  - or

    当多个条件中只要有一个条件满足，可以用关键字 or。例子：

    ```
    print((1 <= 2) or (2 <= 1)) # True
    ```

    只有所有条件都不满足，使用 or 的表达式才会为 False。
- 检查列表中的元素

  - in
    如果想要检查某个元素的是否在列表中，可以使用 in 关键字，例子：
    ```

    users = ['Reese', 'Miles', 'Blake']

    print('Blake' in users) # True

    print('Brooke' in users) # False

    ```
  - not in
    not in 就是检查元素不在列表中的关键字，例子：
    ```

    users = ['Reese', 'Miles', 'Blake']

    print('Blake' not in users) # False

    print('Brooke' not in users) # True

    ```

### 2. if 语句

- 简单的 if 语句

  最简单的 if 语句只有一个条件测试和一个操作。

  ```

  if conditional_test:

      do something

  ```

  例子：

  ```

  weather = 'sunny'

  if weather == 'sunny':

      print("It's a fine day today. Let's stay at home") # 会执行这里的代码

  ```

- if-else 语句

  if-else 语句，就是当 if 中条件测试未通过（也就是 False）的时候，就会执行 else 语句中的代码，例子：

  ```

  weather = 'sunny'

  if weather == 'rains':

      print("It's a beautiful day. Let's go out and play")

  else:

      print("It's a fine day today. Let's stay at home") # 会执行这里的代码

  ```

- if-elif-else

  if-elif-else 语句会一次检查每个条件测试，如果遇到通过了的条件测试，会跳过剩余的所有条件测试。例子：

  ```

  age = 18



  if age < 4:

      print('Too young')

  elif age < 20:

      print('Too simple') # 这里的代码会执行

  else:

      print('sometimes naïve')

  ```

  if-elif-else 语句中 elif 的可以有多个，else 也可以省略不写。

### 3. bool()

bool() 方法用于将给定参数转换为布尔类型。这样我们就可以判断一些特殊的值是 False 还是 True 了。例子：

```

print(bool([])) # False

print(bool(0))  # False

print(bool(1))  # True

print(bool(())) # False

```
