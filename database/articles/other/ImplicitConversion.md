最近在面试的时候遇到这样一个问题: **为什么 ++[[]][+[]]+[+[]] = 10？**，纳尼？为什么会有这样的代码，工作中写出来这样的代码怕是要被人打死，然后搜索了一下，居然还真有挺多人遇到的，大概看了一下要解释清楚不是那么容易的，所以用自己能理解的方式总结一下，以后再遇到也好吹一吹。

一步一步来计算：

先算`+[]`，在控制台中直接输入，得到结果：

![](/caisr.github.io/database/images/articles/other/implicit_conversion/image.png)

+[]等于 0，原因是：

在你不知道的 JavaScript（中卷）中有提到一个概念 ToNumber，ToNumber 就是在 js 中需要将非数字值当做数字值来用的时候，那么 js 就会对这个值做 ToNumber 操作。具体是这样操作的：

- ToNumber 基本类型值的转换规则如下：

  - true 转换成 1，false 转换成 0
  - undefinde 转换成 NaN
  - null 转换成 0
  - 字符串转换规则有点多，这里是规范[ToNumber Applied to the String Type](http://es5.github.io/#x9.3.1)

- ToNumber 复杂类型类型值的转换规则如下：

  1. 检查这个值有没有 valueOf 方法，如果有且 valueOf 得到的值是**基本类型**，那么就用 valueOf 得到的值来计算。
  2. 如果第一条不满足，那么就看这个值有没有 toString 方法，如果有且 toString 得到的值是**基本类型**，那么就用 toString 得到的值来计算。
  3. 如果上面条件都没有满足，那么 js 就会报错。

按照上面的步骤来看：

![](/caisr.github.io/database/images/articles/other/implicit_conversion/image1.png)

空数组 valueOf 得到的仍然是一个空数组，不是基本类型的值，所以使用的是 toString 返回的空字符串。

那么`+[]`就等于`+''`是多少呢？

根据 mdn 的介绍：

![](/caisr.github.io/database/images/articles/other/implicit_conversion/image2.png)

空字符串转成数字就是 0

经过上面步骤，+[]其实就是+0，结果就是 0

将这个结果代入到`++[[]][+[]]+[+[]]`中就成这样了：

`++[[]][0]+[0]`

由于++的优先级比+高，所以先算左边的`++[[]][0]`，仔细观察这个表达式`[[]][0]`这一块的结果其实就是`[]`，那么`++[[]][0]`就等于`++[]`，可是当你在控制台中输入`++[]`却会报错：

![](/caisr.github.io/database/images/articles/other/implicit_conversion/image3.png)

和这个是一样的

![](/caisr.github.io/database/images/articles/other/implicit_conversion/image4.png)

详细原因看这里[从++[[]][+[]]+[+[]]==10?深入浅出弱类型 JS 的隐式转换](https://github.com/jawil/blog/issues/5)

所以`++[]`可以转换成

```
let result = []
++result
```

`++result`等于`result = result + 1`，那么这里就可以用到 ToNumber 的规则了：

`[] + 1` 用 ToNmuber 转换就是`0 + 1`，所以`++[[]][0]`的结果就是 1

再将这个结果代入到上面的表达式中就是

`1 + [0]`，还是这样的套路，将[0]用 ToNumber 规则转换：

![](/caisr.github.io/database/images/articles/other/implicit_conversion/image5.png)

`1 + [0]` 就等于 `1 + '0'`

在 js 中一个数字加一个字符串，最后的结果就是将这两个值拼接成一个字符串。

![](/caisr.github.io/database/images/articles/other/implicit_conversion/image6.png)

可是有这么一种情况：

![](/caisr.github.io/database/images/articles/other/implicit_conversion/image7.png)

为什么 `{} + 1`等于 1 啊，不应该是'[object Object]1'吗？

我的理解是这样的，当`{}`在前面的时候，js 认为`{}`就是一个代码块，而不是一个空对象。就像这样：

![](/caisr.github.io/database/images/articles/other/implicit_conversion/image8.png)

那么上面的代码转换一下就是：

```
{ console.log(2) }; +2
```

但是不知道这个理解对不对。
