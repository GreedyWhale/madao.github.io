lambda 表达式，简单来说就是匿名函数。它可以简化函数的写法，例子：

```
def add(a, b):
    return a + b

# lambda表达式写法

add1 = lambda a, b: a + b

print(add(1,2)) # 3
print(add1(1,2)) # 3
```

使用 lambda 表达式不用写函数名，不用写 return，`:`前是参数，`:`后是返回值。

![](/caisr.github.io/database/images/articles/python/lambda/image.png)
