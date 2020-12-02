this 是一个比较难理解的关键字，它在不同情况下的指向不同，而且它字面意思也很容易让人误解，因为它并不指向函数本身。

首先理解 this，要记住以下两点：

1. this 不指向函数本身
2. this 不指向函数的作用域

例子：

```

// this不指向函数本身



function foo(num) {

  console.log(`foo被调用了${num}次`)

  this.count++

}

foo.count = 0



for(let i = 1; i < 6; i++) {

  foo(i)

}



console.log(foo.count)

```

上面代码的运行结果是这样的

![image.png](/madao.github.io/database/images/articles/javascript/this/image.png)

可以看出，foo 被调用了 5 次，但最后打印出的 foo.count 却是 0，这就证明函数 foo 中的 this.count 并不指向它自己的 count。

```

// this不指向函数的作用域

function foo() {

  let a = 1

  console.log(this.a)

}

foo() // undefined

```

上面代码得到是 undefined，说明 this.a 并不是函数 foo 作用域中声明的变量 a，所以 this 也不指向函数的作用域

- ### this 是什么

  this 是执行上下文中的一个属性，其实我一直不懂上下文这个词，所以我把执行上下文想象成一个对象，这个对象中有一个属性叫做 this。每当函数被调用的时候都会创建一个执行上下文。除了函数在被调用的时候都创建的执行上下文，还有全局执行上下文（函数之外的执行上下文），全局执行上下文中的 this 指向*全局对象*（浏览器中是 window），但是严格模式下全局执行上下文中的 this 指向的是 undefined

- ### 如何确定 this 的指向

  this 的指向由函数的调用方式决定，除了箭头函数

  函数调用可以分为以下几种：

  1. 直接调用

     ```

     function foo() {

       console.log(this)

     }

     foo() // Window



     ```

     ```

     'use strict'

     function bar() {

     console.log(this)

     }

     bar()  // undefined

     ```

     如果函数是直接调用，它的 this 指向全局对象，在浏览器中就是 window，严格模式下是 undefined

  2. 作为对象的属性调用

     ```

     let obj = {

       bar: 1,

       methods() {

         console.log(this.bar)

       }

     }

     let obj2 = {

       bar: 2,

       obj: obj

     }

     obj.methods()  // 1

     obj2.obj.methods()  // 1

     ```

     当函数作为对象的属性调用的时候，它的 this 指向是最后一个调用它的对象。这里有一个小陷阱：

     ```

     let name = 'rose'

     let obj = {

       name: 'jack',

       sayName() {

         console.log(this.name)

       }

     }

     let foo = obj.sayName

     foo()  // rose

     ```

     foo 相当于直接调用，所以 this 指向全局对象 window

  3. 通过 new 调用

     构造函数只是使用 new 运算符调用的函数，new 做了以下几件事：

     1. 创建一个新对象
     2. 让这个新对象的`__proto__` = 构造函数的 prototype
     3. 将函数调用时的 this 绑定到这个新对象
     4. 如果这个构造函数没有 return 其他对象，那么将这个新对象 return

     例子：

     ```

     function CreatePerson({name = '', gender = ''}) {

       this.name = name;

       this.gender = gender

     }

     CreatePerson.prototype = {

       constructor: CreatePerson,

       getName() {

         console.log(this.name)

       },

       getGender() {

         console.log(this.gender)

       },

       work() {

         console.log('work')

       },

       eat() {

         console.log('eat')

       },

       sleep() {

         console.log('sleep')

       },

       play() {

         console.log('play')

       }

     }



     const Tom = new CreatePerson({name: 'Tom', gender: 'male'})



     Tom.getName()   //  Tom

     Tom.getGender()  // male

     ```

     看完 new 操作符做的事情后，很容易得出结论，通过 new 调用的函数，this 指向 new 出来的新对象。

  4. 使用 bind，call，apply 方法

     这三种方法都可以改变函数中 this 的指向

     - bind

       bind 方法会返回一个新的函数，这个新的函数的 this 就是 bind 方法的第一个参数，例子：

       ```

       function foo() {

         console.log(this.name)

         console.log(this.age)

       }



       let obj = {

         name: 'alex',

         age: 18

       }



       foo.bind(obj)()  // alex  18

       ```

     - call

       call 方法也能改变 this 的指向，bind 是返回一个新函数，而 call 则是直接调用这个函数。例子：

       ```

       let obj = {

         a: 1,

         b: 2

       }

       function foo()  {

         console.log(this.a)

         console.log(this.b)

       }

       foo.call(obj)  // 1, 2

       ```

     - apply

       apply 用法和 call 一样，都可以改变函数 this 的指向，不同点是：

       ```

       fun.call(thisArg, arg1, arg2, ...)

       func.apply(thisArg, [argsArray])

       ```

     call 方法除了确定 this 指向的参数之外的参数是一个一个传入的，而 apply 是数组类型。

  5. 事件处理函数中的 this

     - 使用 addEventListener 添加的事件处理函数中的 this 指向触发事件的元素
     - 内联事件处理函数中的 this 指向监听器所在的元素

  6. 箭头函数的 this

     箭头函数的 this，并不是在调用的时候确定的，而是在它定义的时候确定的，它的 this 指向的是它外层函数的 this，如果外层是全局执行上下文，那么它指向的就是全局对象。

     例子：

     ```

     var a = 1

     const foo = () => {console.log(this.a)}

     const obj = {

       a: 2,

       methods() {

         return () => {console.log(this.a)}

       }

     }

     const obj1 = {

       a: 3,

       methods: () => {console.log(this.a)}

     }

     foo()  // 1

     obj.methods()()  // 2

     obj1.methods()   // 1

     ```

以上就是确定 this 指向的几种情况了。

最后推荐一篇文章[this 的值到底是什么？一次说清楚](https://zhuanlan.zhihu.com/p/23804247)，可以在不用记这么多状态下，确定 this 的指向。
