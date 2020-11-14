列表是由一系列按特定顺序排列的元素组成，Python 中用`[]`表示列表。例子:

```
languages = ['JavaScript', 'Python', 'Java', 'c']
```

列表的索引从 0 开始，如果想要访问列表中的某个元素，只需要使用对应的索引即可，例子：

```

languages = ['JavaScript', 'Python', 'Java', 'c']



print('My favorite language is ' + languages[1]) # My favorite language is Python

```

还可以使用负数来获取列表中的元素，比如-1 就代表列表最后一个元素，-2 就是倒数第二个，以此类推。例子：

![](/madao.github.io/database/images/articles/python/list/image.png)

### 1. 修改列表元素

修改列表元素，只需要找到这个元素，重新赋值即可，例子：

![](/madao.github.io/database/images/articles/python/list/image1.png)

### 2. 添加元素

- append()

  append()方法可以将一个元素添加到列表的末尾。例子：

  ![](/madao.github.io/database/images/articles/python/list/image2.png)

- extend()

  extend()方法可以将多个元素添加到列表末尾。例子：

  ![](/madao.github.io/database/images/articles/python/list/image3.png)

- insert()

  insert()方法可以在列表的任何位置添加元素。例子：

  ![](/madao.github.io/database/images/articles/python/list/image4.png)

### 3. 删除元素

- del

  del 语句可以删除列表中的元素，前提是你要知道被删除元素的索引，例子：

  ![](/madao.github.io/database/images/articles/python/list/image5.png)

- pop()

  pop()方法可以将列表中任意位置的元素删除，它会返回被删除的元素，如果不传参数，它会将列表的最后一项删除。例子：

  ![](/madao.github.io/database/images/articles/python/list/image6.png)

- remove()

  当不知道元素的索引时，可以使用 remove()方法根据值删除元素。例子：

  ![](/madao.github.io/database/images/articles/python/list/image7.png)

  需要注意的是 remove()方法只会删除找到的第一个值，也就是列表中如果有重复的元素，只会删除第一个，例子：

  ![](/madao.github.io/database/images/articles/python/list/image8.png)

### 4. 列表排序

- sort()

  sort()方法会将列表按照首字母顺序或者数字从小到大排序，这个方法会改变原列表，例子：

  ![](/madao.github.io/database/images/articles/python/list/image9.png)

  可以传入 reverse = True 将列表按照字母或者数字从大到小，例子：

  ![](/madao.github.io/database/images/articles/python/list/image10.png)

- sorted()

  sort()方法会改变原列表，sorted()不会，它会返回一个新的列表，例子：

  ![](/madao.github.io/database/images/articles/python/list/image11.png)

  sorted()同样可以接受 reverse 参数

- reverse()

  reverse()方法可以将列表倒着排序，它会改变原列表，例子：

  ![](/madao.github.io/database/images/articles/python/list/image12.png)

### 4. 列表的长度

len()方法可以获得列表的长度。例子：

![](/madao.github.io/database/images/articles/python/list/image13.png)

长度是从 1 开始算的，如果获取一个列表中不存在的索引，会报错。例子：

![](/madao.github.io/database/images/articles/python/list/image14.png)

### 5. 元组

元组是不可变的列表，Python 中使用()来表示，例子：

```
colors = ('red', 'green', 'blue')
```

由于它是不可变的，所以不能对元组进行排序，添加和删除。
元组是可以比较大小的，例子：

```

my_tuple = ((1,2), (3,4))



print(my_tuple[0] < my_tuple[1]) # True

```

它的比较是过程是：

```
(1 + 2) < (3 + 4)
```

### 6. 切片

如果想从列表中一次性通过索引获取多个元素，这就叫做列表的切片。例子：

```

numbers = [1,2,3,4,5,6,7,8,9]

print(numbers[1:7])

```

![](/madao.github.io/database/images/articles/python/list/image15.png)

它不会改变原列表。

- 三种简写
  1. 省略结尾。
     ```

     numbers = [1,2,3,4,5,6,7,8,9]

     print(numbers[1:]) # [2, 3, 4, 5, 6, 7, 8, 9]

     ```
     这表示从列表的第一开始一直到列表的最后一项。
  2. 省略开头。
     ```

     numbers = [1,2,3,4,5,6,7,8,9]

     print(numbers[:5]) # [1, 2, 3, 4, 5]

     ```
     这表示从列表的第一项开始一直到索引 4。这里注意的一点是这里返回的元素不包括索引结束的那一项
  3. 全部省略。
     ```

     numbers = [1,2,3,4,5,6,7,8,9]

     print(numbers[:]) # [1,2,3,4,5,6,7,8,9]

     ```
     全部省略相当于返回整个列表
  索引值也是可以用负数的，例子：
  ![](/madao.github.io/database/images/articles/python/list/image16.png)

### 7. for 语句遍历列表

在 Python 中可以使用`for`语句进行列表的遍历，例子：

```

animals = ['tiger', 'dog', 'cat', 'lion', 'leopard']

for animal in animals:

    print(animal)

```

![](/madao.github.io/database/images/articles/python/list/image17.png)

Python 中通过缩进判断代码块，也就是说 for 后面缩进的代码都是每次迭代的时候需要执行的代码，注意 for 语句后面是有一个`:`号的。例子：

```

animals = ['tiger', 'dog', 'cat', 'lion', 'leopard']

for animal in animals:

    print(animal)

    print('Traversing')

print('End of traversal')

```

![](/madao.github.io/database/images/articles/python/list/image18.png)

可以看到上面代码中每次迭代都打印了两句话。当遍历结束后会打印出 End of traversal，因为`print('End of traversal')`这一句代码没有缩进，不算是 for 语句的代码块。

在 Python 中的缩进有一个惯例：总是将代码块缩进 4 个**空格**，这很重要。

### 8. 数字列表

- range()

  range()方法会生成一系列的数字，例子：

  ```

  for value in range(1, 10):

      print(value)

  ```

  ![](/madao.github.io/database/images/articles/python/list/image19.png)

  它前两个参数就是数字的范围，得到的数字不包括结束的那个参数，用上面的例子说明就是不包括 10。

  range 还接受第三个参数，步长。例子：

  ```

  for value in range(1, 10, 2):

      print(value)

  ```

  ![](/madao.github.io/database/images/articles/python/list/image20.png)

  也就是从 1 开始，每次都加 2。

- list

  将 range 生成的数字变成列表需要使用 list()方法，例子：

  ```

  numbers = list(range(1, 10))

  print(numbers)

  ```

  ![](/madao.github.io/database/images/articles/python/list/image21.png)

  list()方法还可以将元组变成列表，例子：

  ```

  colors = ('red', 'yellow', 'blue')

  print(list(colors))

  ```

  ![](/madao.github.io/database/images/articles/python/list/image22.png)

- max()、min()和 sum()

  - max()，max()可以找出数字列表中最大的值
  - min()，min()可以找出数字列表中最小的值
  - sum()，sum()会对数字列表中的所有数字进行相加，并算出总和。
    例子：

  ![](/madao.github.io/database/images/articles/python/list/image23.png)

### 9. 列表解析

列表解析可以更简洁的生成列表，比如需要一个 1 到 10 的平方的列表，我们可以这样写：

```

squares = []

for value in range(1, 11):

    squares.append(value ** 2)

print(squares) # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

```

用列表解析的写法是这样的：

```

squares = [value ** 2 for value in range(1, 11)]

print(squares) # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

```

最后需要注意的是，如果将一个列表直接赋值给一个变量，当你给这个变量新增元素或删除元素的时候都会影响到原来的列表，例子：

```

teacher_names = ['Bryce', 'Brynn', 'Max', 'Cole', 'Kate']

student_names = teacher_names

student_names.append('Juan')

print('teacher_names:', teacher_names)

print('student_names:', student_names)

```

![](/madao.github.io/database/images/articles/python/list/image24.png)

这一点和 JavaScript 中的引用类型一样。

如果不想改变原列表，那么只要用切片就好了，例子：

```

teacher_names = ['Bryce', 'Brynn', 'Max', 'Cole', 'Kate']

student_names = teacher_names[:]

student_names.append('Juan')

print('teacher_names:', teacher_names)

print('student_names:', student_names)

```

![](/madao.github.io/database/images/articles/python/list/image25.png)
