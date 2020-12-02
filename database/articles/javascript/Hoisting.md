> JavaScript 代码在执行前会将所有的声明移动到各自作用域的最顶端，这个过程叫做提升。

### 1. 需要记住的几条规则：

- 函数声明优先提升，然后才是变量声明。

- 重复的变量声明会被忽略。

- 重复的函数声明，后面的声明会覆盖前面的声明。

- let 和 const 声明的变量存在暂时性死区（temporal dead zone，简称 TDZ）

  > 使用 let 或 const 命令声明变量之前，该变量都是不可用的。

### 2. 例子：

- 变量声明提升

  ```

  console.log(bar)   // undefined

  var bar = 1

  console.log(bar)  // 1



  /*提升后*/



  var bar

  console.log(bar)

  bar = 1

  ```

- 函数声明提升

  ```

  foo()   // fuck oriented object

  function foo() {

    console.log('fuck oriented object')

  }



  /*提升后*/



  function foo() {

    console.log('fuck oriented object')

  }

  foo()

  ```

- 函数表达式提升

  ```

  foo()   // 报错 ： foo is not a function

  var foo = function () {

    console.log('fuck oriented object')

  }



  /*提升后*/



  var foo

  foo()

  foo = function () {

    console.log('fuck oriented object')

  }

  ```

- 函数声明优先

  ```

  foo()   // fuck oriented object

  var foo = 1

  function foo() {

    console.log('fuck oriented object')

  }

  console.log(foo)  // 1



  /*提升后*/

  function foo() {

    console.log('fuck oriented object')

  }

  var foo

  foo()

  foo = 1

  ```

- 重复的变量声明会被忽略

  ```

  foo()  // 2

  var foo

  function foo() {

    console.log(1)

  }

  foo = function () {

    console.log(2)

  }



  /*提升后*/



  function foo() {

    console.log(1)

  }

  foo()

  foo = function () {

    console.log(2)

  }

  ```

- 重复的函数声明，后面会覆盖前面的

  ```

  fn()  // 2

  function fn() {

    console.log(1)

  }

  function fn() {

    console.log(2)

  }

  ```

- let 和 const 命令声明的变量存在暂时性死区

  ```

  console.log(bar)   //  报错 bar is not defined

  const bar = 1

  ```

  ```

  console.log(foo)  // 报错 foo is not defined

  let foo = 2

  ```

至于 let 和 const 声明的变量存不存在提升推荐看下这篇文章：
[我用了两个月的时间才理解 let](https://zhuanlan.zhihu.com/p/28140450)
