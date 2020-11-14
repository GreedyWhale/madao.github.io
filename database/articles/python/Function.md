函数可以理解为带名字的代码块，用于完成具体的任务。当出现两次或两次以上的重复代码块的时候，就需要考虑将这些代码块封装成函数，当然不重复出现的代码块也可以写成函数。

### 1. def

Python 中函数通过 def 关键字定义，例子：

```
def sayHello():
    print('Hello !')


sayHello() # Hello !
```

调用的时候使用函数名加括号。

### 2. 实参和形参

函数是可以接受调用者传递给它的信息，这些信息称为参数。例子：

```
def sayName(username):
    print('Hello, ' + username.title() + '!')


sayName('allen') # Hello, Allen!
```

例子中的 username 叫做 - **形参**，调用时传入的 allen 叫做 - **实参**

### 3. 位置实参和关键字实参

- 位置实参

  位置实参就是传入的实参，要和函数定义时的形参位置一一对应。例子：

  ```
  def collect_userinfo(name, age):
      print('Hello!', name.title())
      print('Your age is', age)


  collect_userinfo('jesse', 12)
  # Hello! Jesse
  # Your age is 12

  # 将实参位置调换
  collect_userinfo(12, 'jesse')
  # 报错，因为name被传入的一个数字，数字是没有title()方法的。
  # Traceback (most recent call last):
  # AttributeError: 'int' object has no attribute 'title'
  ```

- 关键字实参

  关键字实参就是不用管形参的位置，只需要传入`key = value`，这种形式的实参就可以，例子：

  ```
  def collect_userinfo(name, age):
      print('Hello!', name.title())
      print('Your age is', age)

  collect_userinfo(age=12, name='jesse')
  # Hello! Jesse
  # Your age is 12
  ```

  从例子中看出，age 和 name 的位置和形参是不一样的，但是用`key = value`这种形式传入，函数是正常执行的，key 就是形参的名字。

### 4. 参数的默认值

可以在定义函数的时候，可以给每个形参指定默认值，如果调用的时候给形参传入了实参，那么就使用这个实参，如果没有那么就会用该形参的默认值，例子：

```
def collect_userinfo(name, age, gender='male'):
    print('Hello!', name.title())
    print('Your age is', age)
    print('Your gender is', gender)

collect_userinfo('allen', 100)
# Hello! Allen
# Your age is 100
# Your gender is male
```

例子中调用函数的时候，并没有传入 gender 形参的实参，但是函数还是正常的执行了。

所以，以下写法调用函数都是等效的：

```
def collect_userinfo(name, age, gender='male'):
    print('Hello!', name.title())
    print('Your age is', age)
    print('Your gender is', gender)


collect_userinfo('allen', 12, 'male')
collect_userinfo(age=12, name='allen', gender='male')
collect_userinfo(name='allen', age=12)
```

参数可以是任意类型的值。

### 5. 返回值

函数也可以返回一些值，比如说格式化一些数据，然后将格式化后的数据返回出来，以便使用，函数返回值可以是任意类型的，例子：

```
def get_formatted_user_info(name, age, gender):
    return {
      name: name.title(),
      age: age,
      gender: gender
    }


user_info = get_formatted_user_info('allen walker', 19, 'male')
print(user_info)
# {'allen walker': 'Allen Walker', 19: 19, 'male': 'male'}
```

- 返回多个值

  函数中可以返回多个值，例子：

  ```
  def get_position(x, y):
      return x * 100, y * 100

  x, y = get_position(10, 15)
  print(x, y) # 1000 1500
  ```

  函数返回多个值其实返回的一个元组：

  ```
  def get_position(x, y):
      return x * 100, y * 100

  postion = get_position(10, 15)
  print(postion) # (1000, 1500)
  ```

  如果，没有 return 语句，函数会返回 None

### 6. 列表参数

当参数是一个列表的时候，需要注意在函数中对其进行的修改，都会使原始列表改变，例子：

```
list = [1, 2, 3, 4, 5]


def get_last_item(list):
    return list.pop()


last_item = get_last_item(list)
print(last_item, list) # 5 [1, 2, 3, 4]
```

如果不想改变原始列表，那么可以使用切片向函数中传递原始列表的副本，例子：

```
list = [1, 2, 3, 4, 5]


def get_last_item(list):
    return list.pop()


last_item = get_last_item(list[:])
print(last_item, list) # 5 [1, 2, 3, 4, 5]

```

- 任意数量的参数

如果一个函数的参数是不确定数量的，那么可以用`*args`的方式定义形参，例子：

```
def make_pizza(*toppings):
    print(toppings)

make_pizza('mushrooms', 'green peppers', 'extra cheese') # ('mushrooms', 'green peppers', 'extra cheese')
```

例子中将 toppings 打印出来，它是一个元组。形参中`*`的星号让 Python 创建一个空的元组，然后把接受到的值都放进这个元组中。除了例子中的那种传递方式，也可以这样传递：

```
def make_pizza(*('mushrooms', 'green peppers', 'extra cheese'))
```

如果想要传递不确定数量的关键字实参，可以用`**kv`形式，例子：

```
def set_user_info(name, **user_info):
    print(user_info)

set_user_info('Jack', age=12, address='hell') # {'age': 12, 'address': 'hell'}
```

形参中`**`会让 Python 创建一个空的字典，然后接受到的值都放进这个字典中，当然它也可以这样传递：

```
set_user_info('Jack', **{'age': 12, 'address': 'hell'}) # {'age': 12, 'address': 'hell'}
```

### 8. 模块

函数的优点是之一就是可以与主程序分类，将函数放在单独的模块中，使用的时候导入即可，模块就是一个以`.py`为扩展名的文件。

- 导入

  导入模块中的函数需要用到 import 关键字

- 导入整个模块

  创建一文件叫`print.py`

  ```
  # print.py

  def say_something(string):
      print(string)
  ```

  再创建一个叫`hello.py`的文件

  ```
  # hello.py

  import print

  print.say_something('hello') # hello
  ```

  当导入整个模块的时候，就可以用`module_name.function_name()`这种形式调用模块里的函数。

- 导入特定函数

  如果只想要模块中某个函数可以用`from module_name import function_name`这种方式，例子：

  ```
  # a.py

  def x():
      print('x')


  def y():
      print('y')


  def z():
      print('z')

  # b.py

  from a import x

  x() # x
  ```

  当用这种方式导入模块内的函数时，调用的时候就不用带上模块名了，直接通过函数名调用。

  也可以一次性导入多个函数：

  ```
  # b.py

  from a import x, y ,z
  ```

- 导入所有函数

  ```
  from module_name import *
  ```

  导入所有函数的写法是用`*`，这样就可以导入模块中的所有函数，调用时直接使用函数名即可

- 指定别名

  使用`as`关键字可以给模块或者函数指定别名，例子：

  ```
  # 给模块指定别名
  from module_name as mn

  mn.xxx()

  # 给函数指定别名

  from module_name import function_name as fn

  fn()
  ```
