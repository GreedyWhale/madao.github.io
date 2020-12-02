有时候我们希望给一个函数新增一些功能，但是又不想对原有函数进行修改，那么这时候就用到了装饰器或者说是装饰模式。

例子：

```

# 1. 有一个求和函数sum

def sum(a, b):

    return a + b



# 2. 如果想要知道这个函数的执行时间，如何实现?



# 实现一

import time



def sum(a, b):

    return a + b

start = time.time()

sum(5, 6)

end = time.time()



print('sum函数的执行时间大约是：%f秒' % (end - start))



# 3. 如果还有一个求差函数sub，要知道这个函数的执行时间，如何实现?

'''

按照实现获取sum函数执行时间的思路，完全可以照搬上面的代码，之前说过有重复

地方尽量封装成函数。

'''



# 实现二



import time





def print_execution_time(func):

    def wrapper(*args, **kwargs):

        start = time.time()

        result = func(*args, **kwargs)

        end = time.time()

        print('%s函数的执行时间大约是：%f秒' % (func.__name__, end - start))

        return result

    return wrapper





def sum(a, b):

    return a + b





def sub(a, b):

    return a - b





print_execution_time(sum)(10, 20)

print_execution_time(sub)(10, 20)



```

这样就实现了一个函数用于获取某个函数的执行时间，
这种模式就是装饰模式，有点像闭包，只不过传递的参数
换成了传递函数。

但是这样做有点问题，就是原本函数的调用方式改变了。而且函数主要的目的也变得
很模糊，原函数主要是用于计算，改写成这样后就好像获取执行时间才是主要目的。

Python 提供了一个语法糖 - 装饰器@。通过这个可以写出更为简洁的代码。例子：

```

# 上面例子使用Python提供的装饰器改写：



import time





def print_execution_time(func):

    def wrapper(*args, **kwargs):

        start = time.time()

        result = func(*args, **kwargs)

        end = time.time()

        print('%s函数的执行时间大约是：%f秒' % (func.__name__, end - start))

        return result

    return wrapper



@print_execution_time  # 新增

def sum(a, b):

    return a + b



@print_execution_time  # 新增

def sub(a, b):

    return a - b



sum(10, 10)

sub(1, 3)

```

结果：

![](/madao.github.io/database/images/articles/python/decorator/image.png)

用了装饰器之后，调用和以前完全一样。只是在定义函数的时候加上`@装饰函数名`这种语法，原本函数就获得了装饰函数中新增的功能。

上面例子中我们都用了`func.__name__`，这句代码的意思是获取函数的名字。但是用了上面装饰器之后我们再看一下 sum 和 sub 函数的`__name__`属性：

```
print('sum的__name__是：%s' % (sum.__name__))
print('sub的__name__是：%s' % (sub.__name__))
```

![](/madao.github.io/database/images/articles/python/decorator/image1.png)

变了，全部变成 wrpper 了。这是因为使用`@print_execution_time`的时候实际上等于
`sum = print_execution_time(sum)`，得到结果就是 wrapper 函数。我们可以使用 funcools 模块中的 wrap 方法来改正这个问题：

```

import time

from functools import wraps # 新增





def print_execution_time(func):

    @wraps(func)                  # 新增

    def wrapper(*args, **kwargs):

        start = time.time()

        result = func(*args, **kwargs)

        end = time.time()

        print('%s函数的执行时间大约是：%f秒' % (func.__name__, end - start))

        return result

    return wrapper





@print_execution_time

def sum(a, b):

    return a + b





@print_execution_time

def sub(a, b):

    return a - b





sum(10, 10)

sub(1, 3)



print('sum的__name__是：%s' % (sum.__name__))  # sum的__name__是：sum

print('sub的__name__是：%s' % (sub.__name__))  # sub的__name__是：sub

```

装饰器本身也是可以带参数的，带参数的装饰器写法如下：

```

import time

from functools import wraps





def print_execution_time(text):

    def decorator(func):

        @wraps(func)

        def wrapper(*args, **kwargs):

            start = time.time()

            result = func(*args, **kwargs)

            end = time.time()

            print('%s函数的执行时间大约是：%f秒' % (func.__name__, end - start))

            print(text)

            return result

        return wrapper

    return decorator





@print_execution_time('今天天气不错')

def sum(a, b):

    return a + b





@print_execution_time('今天天气不好')

def sub(a, b):

    return a - b





sum(10, 10)

sub(1, 3)



print('sum的__name__是：%s' % (sum.__name__))

print('sub的__name__是：%s' % (sub.__name__))



```

结果：

![](/madao.github.io/database/images/articles/python/decorator/image2.png)

写法就是在原有的装饰器上再包裹一层函数。

`@print_execution_time('今天天气不错')` 相当于

`sum = print_execution_time('今天天气不错')(sum)`

总结下两种写法：

1. 不带参数的装饰器

   ```

   from functools import wraps





   def decorator(func):

       @wraps(func)

       def wrapper(*args, **wkargs):

           # 写新增功能

           result = func(*args, **wkargs)

           return result

       return wrapper

   ```

2. 带参数的装饰器

   ```

   from functools import wraps


   def decorator_wrapper(*args, **wkargs):

       def decorator(func):

           @wraps(func)

           def wrapper(*sub_args, **sub_wkargs):

               # 写新增功能

               result = func(*sub_args, **sub_wkargs)

               return result

           return wrapper

       return decorator

   ```
