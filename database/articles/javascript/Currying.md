今天开始学习饥人谷方方老师的 JS 深入浅出，总结下之前不知道的知识点。

先推荐一个网站，可以清晰的看到函数执行过程中，调用栈的变化[latentflip](http://latentflip.com/loupe/)。

- ### 柯里化

  > 柯里化是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。  
  > -----[柯里化](https://zh.wikipedia.org/wiki/%E6%9F%AF%E9%87%8C%E5%8C%96)

  由于我在实际的工作中基本没有用到柯里化，所以看了视频和网上的一些文章后，还是不太理解这样做的好处，但是这部分的知识我又是不知道的，所以先记下来，等以后用到了说不定就理解了。先无耻的将别人总结的柯里化的作用抄袭过来：

  1. 延迟计算
  2. 参数复用

- ### 如何将一个函数柯里化

  例子 1：

  ```

  function sum(x,y) { return x + y; }



  如何将上面的函数变成下面这种调用方式：



  sum1(1)(2);



  答案：

  function curry(fn) {

      return function(x) {

          return function (y) {

              return fn.call(undefined, x, y);

          }

      }

  }



  let sum1 = curry(sum);



  sum1(1)(2);

  ```

  例子 2：

  ```

  function sum(x,y,z) { return x + y + z; }



  如何将上面的函数变成下面这种调用方式：



  sum1(1)(2)(3);



  答案：

  function curry(fn) {

      return function(x) {

          return function (y) {

              return function(z) {

                  return fn.call(undefined, x, y, z);

              }

          }

      }

  }



  let sum1 = curry(sum);



  sum1(1)(2)(3);

  ```

  通过上面两个例子可以发现，把一个函数柯里化的方法就是，函数接受一个函数，返回新函数，需要被柯里化的函数的参数个数，就是返回新函数的个数，当返回的次数和需要被柯里化的函数的参数个数相同或大于时，就执行需要被柯里化的函数。

  我们不能像例子中那样去写代码，不然肯定要被人打死，如何写一个通用的方法？

  通过例子总结出三个关键点：

  1.  需要被柯里化函数的参数个数
  2.  已固定的参数个数
  3.  已固定的参数个数 是否等于或大于 需要被柯里化函数的参数个数

  ```

  function curry(fn, fixedParmas) {

      if(!Array.isArray(fixedParmas)) {

          fixedParmas = []

      }

      return function(...rest) {

          if(fixedParmas.length + rest.length < fn.length) {

              return curry(fn, fixedParmas.concat(rest))

          }

          return fn.apply(undefined, fixedParmas.concat(rest))

      }

  }



  function sum(x,y,z){ return x + y + z }



  let sum1 = curry(sum);



  sum1(1);

  sum1(1)(2);

  sum1(1)(2)(3);  // 6



  sum1(10, 100)(2);  // 112

  ```

  解释一下上面函数：

  ```

   curry 函数接受两个参数 fn， fixedParmas。

     1. fn就是需要被柯里化的函数，

     2. fixedParmas是需要被固定的参数，可以不传



   if(!Array.isArray(fixedParmas)) {

      fixedParmas = []

   }

     1.这里是用来兼容不传固定参数。因为有可能这样调用

       curry(sum) 或者 curry(sum，[1]), fixedParmas这个数组也用来存放已固定

       的参数



   return function(...rest) {

       if(fixedParmas.length + rest.length < fn.length) {

           return curry(fn, fixedParmas.concat(rest))

       }

       return fn.apply(undefined, fixedParmas.concat(rest))

   }



   1. ...rest是es6的语法，因为arguments是一个类数组对象，要转换成数组才能用

   concat合并。而且MDN上说对参数使用slice会阻止某些JavaScript引擎中的优化

   (比如 V8)。...rest就是新的固定参数。

   2. 判断已固定的参数 + 新的固定参数是否小于需要被柯里化的函数的参数。

   函数的参数个数可以用 length 属性获得，所以fn.length就是

   需要被柯里化的函数的参数，如果小于那么继续柯里化（递归）。直到已固定的参

   数大于或等于需要柯里化的函数的参数。最后执行需要柯里化的函数的参数。

  ```

  我觉得就是递归那里可能有些难理解。看下效果：

  ![](/caisr.github.io/database/images/articles/javascript/curry/image.png)

  从上图看出，当传入的参数不够的时候，是不会计算的，只有等参数个数够了，才会计算，所以作用延迟计算就体现出来了。

  那么参数复用如何体现：

  ![](/caisr.github.io/database/images/articles/javascript/curry/image1.png)

  从图中看出，固定了一个参数后，之后只要传一个参数就可以了。
