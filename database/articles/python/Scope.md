Python 有局部作用域和全局作用域。还可以细分为这四种情况：

- L（local），局部作用域
- E(enclosing)，嵌套作用域
- G(global)，全局作用域
- B(built-in)，内置作用域

### 一. L（local），局部作用域

如果一个变量在函数内部，那么这个变量就叫做局部变量，函数之外是无法访问的，例子：

```
def say_name():
    name = 'Allen'
    print(name)
say_name()
print(name)
```

![](/madao.github.io/database/images/articles/python/scope/image.png)

函数在执行完毕后，会将它内部的变量回收（删除）

如果想要在函数外部使用函数内部的变量。可以使用 golbal 关键字

```
def say_name():
    global name
    name = 'Allen'
    print(name)
say_name()
print(name)
```

![](/madao.github.io/database/images/articles/python/scope/image1.png)

### 二. E(enclosing)，嵌套作用域

作用域之间是可以相互嵌套的，比如两个函数之间嵌套。里面的函数可以访问它外部函数作用域中的变量，但是外面的函数无法访问内部函数的作用域，例子：

```
def foo():
    a = 1
    def bar():
        b = 2
        print(a)
    bar()
    print(b)

foo()
```

![](/madao.github.io/database/images/articles/python/scope/image2.png)

但是，如果在内部函数中直接重新赋值外部函数作用域中的变量，外部函数作用域中的变量是不会改变的，会得到一个新的局部变量。同样的在函数中改变全局作用域的变量，也是这样。

```
b = 1
def foo():
    a = 1

    def bar():
        a = 2
        print('bar:', a)
    def fn():
        b = 2
        print('fn', b)
    bar()
    fn()
    print('foo:', a)

foo()
print('golbal:', b)
```

![](/madao.github.io/database/images/articles/python/scope/image3.png)

如果想要修改原值，那么就要用到 nonlocal 关键字，例子：

![](/madao.github.io/database/images/articles/python/scope/image4.png)

注意，nonlocal 关键字只针对局部变量，如果寻找的变量处于全局作用域中，会报错，，例子：

```
a = 1

def foo():
    nonlocal a
    a = 2
    print('foo:', a)

foo()

```

![](/madao.github.io/database/images/articles/python/scope/image5.png)

如果 nonloacl 关键字在上一层中找到了变量，那么就会停止寻找，如果没有就继续向上直到全局作用域：

```
def foo():
    a = 1
    def bar():
        print('bar:', a)
        def fn():
            nonlocal a
            a = 2
            print('fn:', a)
        fn()
    bar()
    print('foo:', a)

foo()
```

![](/madao.github.io/database/images/articles/python/scope/image6.png)

### 三. G(global)，全局作用域

全局作用域就是模块最顶层声明的变量。每一个.py 结尾的文件就是一个模块，每个模块都有不同的全局作用域，当一个模块导入另一个模块后，在当前模块就可以访问另一个模块的全局作用域中的变量了。全局作用域中的变量，在当前模块的任何地方都可以访问到。

```
# b.py

module_name = 'b'

# a.py

import b

print(b.module_name)
```

![](/madao.github.io/database/images/articles/python/scope/image7.png)

### 四. B(built-in)，内置作用域

内置作用域就是 Python 中内置模块中的变量，比如我们可以直接使用 print 方法而不用导入任何模块。

![](/madao.github.io/database/images/articles/python/scope/image8.png)

Python 中寻找变量的规则会按照 LEGB 规则寻找：

Local -> Enclosing -> Global -> Built-in

最后说一点的是，Python 中没有块级作用域，也就是在类似 for 语句中定义的变量就是全局作用域中的变量：

![](/madao.github.io/database/images/articles/python/scope/image9.png)
