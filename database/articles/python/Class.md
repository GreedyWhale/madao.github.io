我理解的类，就像人类一样，说人类的时候肯定不是指某一个人，而说是所有拥有人基本属性的这些对象。而每个人都有差异，包括性格，身高，体重之类的。Python 中的类也是这样，定义一个类，根据类创建对象，对象都具有一些共有的方法和属性，但是也是可以有自己的方法和属性，根据类创建对象被称为实例化。

### 1. 类

Python 中类通过关键字 class 创建。例子：

```

class Human():

    def __init__(self, name, age):

        self.name = name

        self.age = age

    def cry(self):

        print("I'm " + self.name + ' ,I am so sad')

    def laugh(self):

        print("I'm " + self.name + ' ,I am very happy')

```

例子中简单的定义了一个类，类的名字是 Human，注意类的命名，一般类的命名都是驼峰命名，而且首字母大写。

- `__init__`

  `__init__()`是一个特殊的方法，每当使用类创建新的实例的时候，Python 都会自动的执行它，它前后各有两个下划线`__`，这是一种约定，避免和普通的方法命名冲突，所以我们自己定义的变量，最好不要用`__xx__`来命名。

  `__init__`方法有中第一个形参必须是 self，它指向被创建的实例（和 JavaScript 中的 this 一样），但是 self 不用传递，它会自动传递，用上面的 Human 举例：

  ```
  alex = Human('alex', 20)

  print(alex.name, alex.age) # alex 20
  ```

  只需要给 self 后面的形参传递值即可。

  同样的 cry 和 laugh 方法中的 self 也是不用传递的

- 创建实例

  创建实例的方法就是调用一下类，然后把类返回的值赋值给一个变量，那么这个变量就会拥有类中定义的共有方法和属性，例子：

  ```

  class Human():

  def __init__(self, name, age):

      self.name = name

      self.age = age

  def cry(self):

      print("I'm " + self.name + ' ,I am so sad')

  def laugh(self):

      print("I'm " + self.name + ' ,I am very happy')

  john = Human('John', 50)

  print(john.name)    # John

  print(john.age)     # 50

  john.cry()          # I'm John ,I am so sad

  john.laugh()        # I'm John ,I am very happy

  ```

  可以根据需求创建任意数量的实例。

- 修改实例的属性

  修改实例属性的值有两种方法：

  - 直接修改

  ```
  # 用上面的Human类

  tom = Human('Tom', 12)

  # 将实例tom的age加1

  tom.age += 1

  print(tom.age) 13
  ```

  - 通过方法修改

  ```

  class Human():

    def __init__(self, name, age):

        self.name = name

        self.age = age

    def cry(self):

        print("I'm " + self.name + ' ,I am so sad')

    def laugh(self):

        print("I'm " + self.name + ' ,I am very happy')

    def addAge(self):

        self.age += 1

  tom = Human('Tom', 12)

  tom.addAge()

  print(tom.age) # 13

  ```

### 2. 继承

类可以另外的类，当一个类继承另一个类的时候，它将自动获得另一个类的方法和属性，原有的类被称为父类，新的类叫做子类。

例子：

```

# 父类

class Human():

   def __init__(self, name, age):

       self.name = name

       self.age = age

   def cry(self):

       print("I'm " + self.name + ' ,I am so sad')

   def laugh(self):

       print("I'm " + self.name + ' ,I am very happy')

   def addAge(self):

       self.age += 1

# 子类

class Superman(Human):

   def __init__(self, name, age):

       super().__init__(name, age)

kal = Superman('Kal', 100)

print(kal.name) # Kal

kal.addAge()

print(kal.age) # 101

```

例子中，Superman 类继承自 Human 类。创建子类时：

1.  父类必须包含在当前文件里，且位于子类之前
2.  定义子类时，必须在括号内指定父类的名称
3.  子类的`__init__`方法中，要使用 super()方法将父类和子类关联起来，调用一下父类的`__init__`方法，注意不要传入 self 参数，其他的参数和父类中`__init__`方法的形参一一对应。

当创建了子类后，通过子类创建的实例会自动的有父类的属性方法。

- 子类的属性和方法

子类也是可以有自己的属性和方法的，例子：

```

# 子类

class Superman(Human):

   def __init__(self, name, age):

       super().__init__(name, age)

       self.address = 'Krypton'

   def fly(self):

       print('I can fly')

kal = Superman('Kal', 100)

print(kal.address) # Krypton

kal.fly() # I can fly

```

- 将实例作为属性

其他类的实例，也是可以作为类的属性的，例子：

```

class Skills():

   def __init__(self):

       self.skills = ['Super Flare', 'Invulnerability', 'Super power', 'Super speed']

   def print_skill_list(self):

       print(self.skills)

class Superman(Human):

   def __init__(self, name, age):

       super().__init__(name, age)

       self.address = 'Krypton'

       self.skills = Skills()

   def fly(self):

       print('I can fly')

kal = Superman('Kal', 100)

kal.skills.print_skill_list() # ['Super Flare', 'Invulnerability', 'Super power', 'Super speed']

```

例子中 Superman 类的 skills 属性是一个类的实例，这样通过 Superman 创建的实例，也可以用到 Skills 类中的方法和属性。

### 3. 导入

类和函数一样，也是可以防止各个模块中，通过 import 关键字导入：

```

# 导入单个类

from module_name import class_name

# 导入多个类

from module_name import class_name_one, class_name_two

# 导入所有类

from module_name import *

```

### 4. Python 标准库

Python 标准库是一组模块，只要安装了 Python 都会包含它。
[文档](https://docs.python.org/zh-cn/3.7/library/index.html)

试试使用 Python 标准库中的 orderedDict 类，orderedDict 类的实例和字典一样，唯一的区别就是它会记录字典键值对添加的顺序。

之前说过字典无序的。所以循环的时候键值对的顺序是不能保证和添加时候的顺序保持一致的。
