### 1. filter

filter 函数用于过滤可迭代对象。它的语法是这样的：

```
filter(function or None, iterable) --> filter object
```

第一个参数是一个函数，第二个参数是一个可迭代对象。filter 函数会根据第一个参数返回 True 还是 False 来决定第二个参数中每一项元素是否保留，它返回的是一个 filter 对象。

例子：

![](/madao.github.io/database/images/articles/python/array_methods/image.png)

filter 对象是一个迭代器，它由惰性运算的特性。如果想要得到一个 list 类型的函数，的借助 list 函数

![](/madao.github.io/database/images/articles/python/array_methods/image1.png)

### 2. map

map 函数的语法是：

```
map(func, *iterables) --> map object
```

我理解的它的作用就是，遍历一个（多个）可迭代对象，然后对这个可迭代对象中的每一个元素都执行传入的函数，并把结果作为最终返回的 map 对象中的元素。

例子：

```

list_1 = [1, 2, 3, 4, 5]

list_2 = [9, 8, 7, 6, 5]

new_list_1 = map(lambda i: i**2, list_1)

new_list_2 = map(lambda x, y: x + y, list_1, list_2)

print(list(new_list_1))  # [1, 4, 9, 16, 25]

print(list(new_list_2))  # [10, 10, 10, 10, 10]

```

从例子中可以看出 map 是可以接受多个可迭代对象的。

### 3. reduce

reduce 不能直接使用，必须从 functools 模块中导入(python3 中)：

```
from functools import reduce

help(reduce)
```

它的语法是这样的:

```
reduce(function, sequence[, initial]) -> value
```

它接受三个参数，第一个是一个函数，第二个是一个可迭代对象，第三个是个可选参数，初始值，reduce 的作用直接看官方解释，我有点不知道怎么说：

```

Apply a function of two arguments cumulatively to the items of a sequence,

from left to right, so as to reduce the sequence to a single value.

For example, reduce(lambda x, y: x+y, [1, 2, 3, 4, 5]) calculates

((((1+2)+3)+4)+5).  If initial is present, it is placed before the items

of the sequence in the calculation, and serves as a default when the

sequence is empty.

```

直接看例子：

```

from functools import reduce

nums = reduce(lambda x, y: x + y, [1,2,3,4,5])

print(nums)

'''

reduce 计算的过程可以理解为下面这样:

(((1 + 2) + 3) + 4) + 5

'''

nums = reduce(lambda x, y: x - y, [1,2,3,4,5])

print(nums)

'''

reduce 计算的过程可以理解为下面这样:

(((1 - 2) - 3) - 4) - 5

'''

```

如果加上 initial 参数就是这样：

```

from functools import reduce

nums = reduce(lambda x, y: x + y, [1,2,3,4,5], 1)

print(nums)

'''

reduce 计算的过程可以理解为下面这样:

((((1 + 1) + 2) + 3) + 4) + 5

'''

nums = reduce(lambda x, y: x - y, [1,2,3,4,5], 1)

print(nums)

'''

reduce 计算的过程可以理解为下面这样:

((((1 -1) - 2) - 3) - 4) - 5

'''

```

### 4. zip

zip 函数的语法是这样的：

```
zip(iter1 [,iter2 [...]]) --> zip object
```

作用直接看例子：

```
zip_example = zip((1,2,3), (3,2,1))
print(list(zip_example)) # [(1, 3), (2, 2), (3, 1)]
```

zip 将两个元组（只要是可迭代对象就可以）合并了，合并对应关系是：

```

(1, 2, 3)

 ↓  ↓  ↓

(3, 2, 1)

    ↓

(1, 3), (2, 2), (3, 1)

```

注意 zip 返回的是 zip 对象，也是一个迭代器，如果想要列表类型，还是得用 list 函数进行转换。

使用 zip 可以轻松的将字典的 key 和 value 对换：

```

dict_a = {

    'a': 'aaa',

    'b': 'bbb',

    'c': 'ccc'

}

zip_example = zip(dict_a.values(), dict_a.keys())

print(dict(zip_example))  # {'aaa': 'a', 'bbb': 'b', 'ccc': 'c'}

```

还有一个用法就是用`*`：

```

zip_example = list(zip((1,2,3), (3,2,1), (3,2,1)))

print(zip_example) # [(1, 3, 3), (2, 2, 2), (3, 1, 1)]

zip_example_restore = list(zip(*zip_example))

print(zip_example_restore) # [(1, 2, 3), (3, 2, 1), (3, 2, 1)]

```

给 zip 的参数加上`*`之后就好像是 zip 逆过程一样。
