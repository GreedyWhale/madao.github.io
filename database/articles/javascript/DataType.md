#### 1. typeof null === 'object'

> 不同的对象在底层都表示为二进制，在 Javascript 中二进制前三位都为 0 的话会被判断为 Object 类型，null 的二进制表示全为 0，自然前三位也是 0，所以执行 typeof 时会返回"object"。

#### 2. 数组的字符串键值如果可以转换成十进制数组，那么就会变成数组的下标。

例子：

![](/madao.github.io/database/images/articles/javascript/dataType/image.png)

#### 3. 函数是可以调用的对象

函数可以像任何其他对象一样具有属性和方法。它们与其他对象的区别在于函数可以被调用，函数的 length 属性时函数形参的数量。

例子：

![](/madao.github.io/database/images/articles/javascript/dataType/image1.png)

#### 4. 类数组对象

类数组对象具有 length 属性，也可以通过`[下标]`这种方法获取对应的值，但是它无法使用数组的 push，forEach，indexOf 等方法，并不是真正的数组。

例子：

![](/madao.github.io/database/images/articles/javascript/dataType/image2.png)

可以通过以下两种方法将类数组对象变为数组：

1. Array.prototype.slice.call(类数组对象)
2. Array.from(类数组对象)

例子：

![](/madao.github.io/database/images/articles/javascript/dataType/image3.png)

#### 5. 0.1 + 0.2 !== 0.3

JavaScript 中数字类型基于 IEEE 754 标准实现，该标准通常被称为“浮点数”，JavaScript 使用的是双精度格式（64 位二进制），二进制浮点数的问题就是：

0.1 + 0.2 !== 0.3;

在二进制浮点数中 0.1 + 0.2 = 0.30000000000000004

可以通过设置一个误差范围来判断 0.1 + 0.2 和 0.3 是否相等。这个误差范围成为“机器精度”，在 JavaScript 中，这个值一般是`2.220446049250313e-16`，从 ES6 开始，可以通过 Number.EPSILON 获得。

```

function numbersCloseEnoughEqual(num1, num2) {

    return Math.abs(num1 - num2) > Number.EPSILON;

}



let a = 0.1 + 0.2;

let b = 0.3;



numbersCloseEnoughEqual(a, b);   // true

```

#### 6. Number.MAX_VALUE，Number.MIN_VALUE，Number.MAX_SAFE_INTEGER，Number.MIN_SAFE_INTEGER

JavaScript 中：

- Number.MAX_VALUE，定义了能够呈现的最大浮点数
- Number.MIN_VALUE，定义了能够呈现的最小浮点数
- Number.MAX_SAFE_INTEGER，定义了能够呈现的最大整数
- Number.MIN_SAFE_INTEGER，定义了能够呈现的最小整数

![](/madao.github.io/database/images/articles/javascript/dataType/image4.png)

#### 7. `.`运算符

`.`运算符需要注意的一点是：因为它是一个有效的数字字符，会被优先认为是数字的一部分，然后才是对象的属性访问运算符，这就导致了下面这些情况：

![](/madao.github.io/database/images/articles/javascript/dataType/image5.png)

#### 8. null 和 undefined

- null，空值
- undefined，未赋值

可以使用 void 运算符获得安全的 undefined，比如：`void 0;`

关于安全的 undefined，可以看看这篇文章[var undefined = 1 这样赋值有效果吗？](https://zhuanlan.zhihu.com/p/22345132)

#### 9. NaN

NaN 指的是“不是一个数字”（not a number）,但是它却是数字类型，而且它是自己和自己不相等：

```

NaN === NaN;  // false



typeof NaN;  // "number"

```

如何检测一个值是不是 NaN：

- isNaN 和 Number.isNaN

  全局方法 isNaN 并不可靠

  ```

  isNaN('foo'); // true

  ```

  foo 并不是 NaN，而结果却是 true。

  所以推荐使用 Number.isNaN 方法：

  ```

  Number.isNaN('foo');  // false

  Number.isNaN(NaN);  // true

  ```

  Number.isNaN 是 ES6 中添加的，ES6 之前可以用下面这种方法检测：

  ```

  if(!Number.isNaN) {

      Number.isNaN = function(value) {

        var n = parseInt(value);

        return n !== n;

    };

  }

  ```

#### 10. 基本类型和引用类型

JavaScript 中除了对象以外，其他类型都叫基本类型。

它们之间的区别通过一个例子说明：

```

let obj = {

    a: 1

};



let obj1 = obj;



obj1.b = 2;



console.log(obj);  {a: 1, b: 2}



console.log(obj1); {a: 1, b: 2}



let str = 1;

let str1 = str;



str1 += 1;



console.log(str);  // 1

console.log(str1); // 2



```

因为引用类型传递的是值的地址，所以 obj 和 obj1 中的地址指向的是同一个对象，改变其中一个会影响到另一个。基本类型相当于将原始值赋值了一份给目标值，原始值和目标值是独立的，相互不会影响。

对于引用类型来说，一个引用无法改变另一个引用的指向。
例子：

```

let arr = [1,2,3];

let arr1 = arr;



arr1 = [4,5,6];



console.log(arr);   // [1,2,3]

console.log(arr1);  // [4,5,6]

```

所以引用类型的值在拷贝的时候分为浅拷贝和深拷贝，浅拷贝就像例子中的一样，直接赋值即可，深拷贝可以使用`JSON.parse(JSON.stringify(obj))`实现。
