我觉得在理解闭包之前，先要理解有两点：

1.  作用域
2.  JavaScript 的垃圾回收机制

作用域就不用多说了，必须懂得基础知识，简单的说下垃圾回收机制：

根据 MDN 的文档，目前主要的垃圾回收算法有两种：

1. 引用-计数，简单的说就是找到所有没有被引用到的对象，把它们回收，但是这个算法存在一个问题，就是无法处理循环引用（a 引用 b，b 引用 a）
2. 标记-清除，标记就是从根对象开始（在浏览器环境中，JavaScript 的根对象就是 window），找所有从根开始引用的对象，再找这些对象引用的对象...，所有能够查询到的对象都不会被清除，查询不到的对象全部被回收。（这也是目前浏览器使用的算法）

垃圾回收的目的是为了释放那些无用的变量占用的内存

参考资料：
[内存管理](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management)
[JavaScript 变量、作用域、垃圾回收机制和闭包理解](https://simmin.github.io/2016/10/10/some-js-concept/#%E4%B8%89%E3%80%81%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86)

了解了垃圾回收机制后，我们就知道在全局变量是不会被回收的（只有页面关闭全局变量才会被回收）。因为在全局作用域中声明一个函数或者变量，相当于给全局对象 window 添加一个属性，例子：

![image.png](/madao.github.io/database/images/articles/javascript/closure/image.png)

现在，来说说闭包。

闭包的特点：

- 闭包是一个函数
- 闭包可以使函数外部的作用域访问函数内部的作用域
- 闭包可以阻止函数在执行完后的内部变量不被回收

例子：

```

function foo() {

  let a = 0

  function fn () {

     a += 1

     console.log(a)

  }

  return fn

}

let bar = foo()

bar()  // 1

bar()  // 2

bar()  // 3

```

函数 foo 返回了一个函数，这个函数由于作用域的原因是可以访问到函数 foo 的作用域的，把这个函数赋值给全局变量 bar，这样 bar 就可以在函数 foo 的作用域外去修改 foo 内部变量 a 的值，由于 bar 是全局变量，所以它不会被回收，而它又引用了 fn 函数，fn 函数又引用了 foo 中的变量 a，导致 foo 在执行完成之后，它内部的变量没有被回收，这就是一个最简单的闭包了。

- ### 闭包的作用

  利用闭包特性，可以实现模块模式，模块模式是一种设计模式，就是将为了实现某个功能的函数或者变量都放在一起。这样做的好处有：

  1. 方便维护
  2. 避免命名冲突
  3. 复用性更高

  例子：

  ```

  function control() {

    let currentSpeed = 0;



    function setSpeed(speed) {

      return currentSpeed = speed;

    }



    function getSpeed() {

      return currentSpeed;

    }



    function accelerate(speed) {

      return currentSpeed += speed;

    }



    function decelerate(speed) {

      return currentSpeed -= speed;

    }



    function getStatus() {

      if (currentSpeed > 0) {

        return 'running';

      } else {

        return 'stop'

      }

    }

    return {

      setSpeed,

      getSpeed,

      accelerate,

      decelerate,

      getStatus

    }

  }



  const car = control()

  ```

  这样就封装了一个简单的模块，该模块暴露的方法有 setSpeed,getSpeed 等等，就可以通过这些方法去改变 control 函数中 currentSpeed 变量。

总结下就是，在函数中通过返回一个新函数，能使函数外部的作用域访问到函数内部的作用域，就产生了闭包。
