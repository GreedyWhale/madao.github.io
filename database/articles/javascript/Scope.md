你一定会对下面这些知识点感到熟悉，因为网络上有很多相似的文章，没错，这些知识都是从《你不知道的 JavaScript（上卷）》中获取的，为啥还要写一遍，因为我记性不好，算是一个读书笔记吧。

### 一. 作用域

> 作用域是一套规则，用于确定在何处及如何查找变量

### 二. 静态作用域 & 动态作用域

静态作用域也叫词法作用域，JavaScript 采用的就是这种作用域，与之对应的就是动态作用域，它们之间的主要区别就是：
**静态作用域是在写代码或者是定义的时候确定的，而动态作用域则是在代码运行时确定的**。

### 三. 静态作用域（词法作用域）

词法作用域就和它的名字一样，是在词法分析阶段确定的，什么是词法分析？JavaScript 代码在执行前需要进行编译，编译分为三个步骤：

1.  词法分析
    将代码分解成词法单元，例如：var a = 1; 会被分解成 var 、a、=、1、;这些词法单元

2.  语法分析
    将词法单元转换成代表程序语法结构的树，这个树称为“抽象语法树”。

3.  代码生成
    将抽象语法树转换为可以执行的代码。

例子：

```

var a = 1

function foo() {

    var a = 2;

    bar();

}



function bar() {

    console.log(a);

}



foo();  // 1

```

### 四. 全局作用域 & 函数作用域 & 块级作用域

- #### 全局作用域
  全局作用域中的变量在整个 JavaScript 代码中的任何地方都可以访问和修改，全局作用域中的变量都是在**全局对象**的属性，如果 JavaScript 代码是运行在浏览器的环境中，那么这个全局对象就是 window，如果是在 node.js 中那么这个全局对象就是 global，例子：
  ```

  var bar = 1;

  function foo() {

    console.log(bar);

  }

  foo();  // 1

  window.foo(); // 1

  window.bar;  // 1

  ```
  这里要注意一下，如果使用 let 和 const 在全局作用域中声明一个变量，这个变量仍然是全局变量，但不是全局对象的属性，例子
  ```

  let a = 1;

  window.a;  // undefined

  function foo() {

    console.log(a);

  }

  foo(); // 1

  ```
- #### 函数作用域

  函数作用域中的变量只能在该函数中访问和修改，在函数之外无法访问和修改，通过闭包可以使函数之外的作用域访问和修改函数作用域中的变量。例子：

  ```

  function foo() {

    var bar = 1;

  }

  foo();

  console.log(bar); // Uncaught ReferenceError: bar is not defined

  ```

  注意：如果不使用任何关键字直接声明一个变量，这个变量为全局变量。例子：

  ```

  function foo() {

      bar = 1

  }

  foo();

  console.log(bar); // 1

  ```

- #### 块级作用域
  块级作用域在 es6 之前就已经存在了，就是 try...catch..语句，例子：
  ```

  try{

    throw 1;

  } catch(a) {

    console.log(a);   // 1

  }

  console.log(a); // a is not defined

  ```
  在 es6 的语法中，用 let 和 const 关键字声明的变量只能在当前代码块中访问和修改。例子：
  ```

  { let foo = 1; }

  { const bar = 2; }

  console.log(foo);  // foo is not defined

  console.log(bar);  // bar is not defined

  ```

### 五. 欺骗词法作用域

使用 eval 和 with 可以欺骗法作用域，但是不推荐这么做，原因是：

1. 严格模式下 with 被禁用
2. 慢
3. eval 会有安全性问题
4. with 有可能声明一些意料之外的变量

例子：

```

// eval

function foo(str, a) {

   eval(str);

   console.log(a, b);

}

var b = 2;

foo('var b = 3', 1);   // 1, 3

// with

function bar() {

 var c = 2;

 with(window) {

   console.log(c);

 }

}

var c = 1;

bar();   // 1

// 测试性能 - eval

let a = 0;

let b = 0;

function useEval() {

 console.time('useEval');

 for(let i = 0; i < 10000; i += 1) {

   a = eval(`${a} + ${i}`);

 }

 console.timeEnd('useEval');

}

function noEval() {

 console.time('noEval');

 for(let i = 0; i < 10000; i += 1) {

   b += i;

 }

 console.timeEnd('noEval');

}

noEval();

useEval();

// 测试性能 - with

let obj = {

 a: 0,

 b: 1

};

function useWith() {

 console.time('useWith');

 with(obj) {

   for(let i = 0; i < 10000; i += 1) {

     a = a + i;

   }

 }

 console.timeEnd('useWith');

}

function noWith() {

 console.time('noWith');

 for(let i = 0; i < 10000; i += 1) {

   obj.b = obj.b + i;

 }

 console.timeEnd('noWith');

}

useWith();

noWith();

```

node.js 中：
![截图](/madao.github.io/database/images/articles/javascript/scope/image.png)

chrome 浏览器控制台中：
![截图](/madao.github.io/database/images/articles/javascript/scope/image1.png)

node.js 中：
![截图](/madao.github.io/database/images/articles/javascript/scope/image2.png)

chrome 浏览器控制台中：
![截图](/madao.github.io/database/images/articles/javascript/scope/image3.png)

- 补充说明

1. 关于 with 有可能声明一些意料之外的变量这一点，直接举个例子说明，因为我感觉用我的表达能力，只会让人感到越来越迷惑：

    ```

    let obj = {

      a: 1

    }

    with(obj) {

      a = 3

      b = 2

    }

    console.log(obj);  //  {a: 3}

    console.log(b);  // 2

    ```

    为啥会这样可以去看下这篇文章[12 种不宜使用的 Javascript 语法](http://www.ruanyifeng.com/blog/2010/01/12_javascript_syntax_structures_you_should_not_use.html)

2. 使用 with 和 eval 变慢的原因，书中有解释，概括下就是引擎在编译的时候会对作用域查找进行优化，以便于在代码运行时快速找到对应的作用域，但是使用的 with 和 eval 的代码，引擎无法明确的知道使用 with 和 eval 的代码的会对作用域做什么修改，所以只能不优化。

### 六. 作用域链 & 作用域查找规则

作用域之间是可以嵌套的，比如：

```

{

   let a = 1;

   {

     let b = 2;

   }

}



function foo() {

   let c = 3;

   function bar() {

     let d = 4;

   }

   bar();

   console.log(d);

}

```

如果执行函数 foo，那么会得到`d is not defined`报错信息，原因就是作用域的查找规则：

作用域查找某个变量会在**当前作用域开始查找**，如果找不到会一层一层向上（或者理解为向外）查找，直到全局作用域。

上面的例子中，foo 函数先从自身的作用域开始查找，找不到就去外层的作用域查找，也就是全局作用域，找完了全局作用域，发现还没有，那么就会停止查找并抛出异常。

而这个查找过程形成的链，就称为作用域链。
