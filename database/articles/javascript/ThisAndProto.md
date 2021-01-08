之前写过一篇文章，总结过 new，this，构造函数及原型。

今天看完饥人谷方方老师的讲的课，有点颠覆我对这些东西的理解。重新总结下：

### 1. 原型链

- 问题 1

  创造 10 个人类对象，它们都有 eat, sleep 方法, 拥有不同的身份证号。

  ```

  let humans = [];

  for(let i = 0; i < 10; i++) {

      let human = {

          id: i,

          eat: function(){},

          sleep: function(){}

      }

      humans.push(human)

  }

  这样就创造了10个人类。

  ```

  但是，这样有一个问题，除了 id 不同，eat，sleep 方法都一样，为什么要创造相同的 eat，sleep 方法 10 次呢，这样是不是浪费了很多内存。

  ```
  // 改进

  let humans = [];

  let common = {

      eat: function(){},

      sleep: function(){}

  }

  for(let i = 0; i < 10; i++) {

      let human = {

          id: i,

          common: common

      }

      humans.push(human)

  }

  这样就节省了很多内存

  ```

  例子中的，eat，sleep 方法就叫做对象的共有属性，它通过 common 属性引用，这个 common 属性对应的就是每个对象的`__proto__`属性，id 就叫做自有属性。

  如图：

  ![](/madao.github.io/database/images/articles/javascript/thisAndProto/image.png)

  在 JavaScript 中，Object.prototype 就是存放所有对象共有属性的的对象。

  ![](/madao.github.io/database/images/articles/javascript/thisAndProto/image1.png)

  `Object.prototype.__proto__` === null ，因为它不再需要引用其他共有属性了，它自己就是为了存放所有对象的共有属性。

- 问题 2

  ```

  let arr = [];

  let obj = {};

  arr.push // function

  obj.push // undefined

  arr.toString //  function

  ```

  数组也是对象，为什么数组有 push 方法，而普通对象没有呢，这种是怎样实现的。

  JavaScript 中除了有 Object 以外，还有 Array（注意首字母都是大写），它们都是函数，用于创建对象和数组（函数也是对象），那么只需要将所有数组的共有属性写在 Array.prototype 中，然后让`被创建的数组.__proto__`指向 Array.prototype，然后`Array.prototype.__proto__` 指向 Object.prototype 就可以让被创建的数组既可以用数组的共有属性，也可以用对象的共有属性了。

  ![](/madao.github.io/database/images/articles/javascript/thisAndProto/image2.png)

  ![](/madao.github.io/database/images/articles/javascript/thisAndProto/image3.png)

  它们的关系是：

  ![](/madao.github.io/database/images/articles/javascript/thisAndProto/image4.png)

  对象的`__proto__`就是对象的原型（可以理解为共有属性），当查找一个对象中的属性的时候，JavaScript 会先查找对象自有属性，没有就去共有属性里找，共有属性里还没有就在**共有属性的共有属性**里继续找，直到 Object.prototype。那么这个过程可以看成

  ```
  let arr = [];
  arr.toString
    ↓
  arr.__proto__ → Array.prototype → Array.prototype.__proto__
                                            ↓
                                    Object.prototype
  ```

  通过 `__proto__ `链接起来的过程就叫做**原型链**

### 2. this

1. 函数的参数只有在传入参数的时候才能确定。
2. this 是函数在调用时，使用 call 方法传入的第一个参数。
3. this 只有在函数传入参数的时候才能确定。

要理解上面三句话必须先理解 call。传送门[Function.prototype.call()
](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)

一般调用函数的时候我们都是这样调用的:

```

function sayHello() {

    console.log('hello')

}

sayHello();

```

这种调用方式其实是一种语法糖

```
sayHello() 可以转换成 sayHello.call(undefined)
```

如果大家都使用 call 来调用函数，那么 this 的值就很好确定了，就是 call 的第一个参数。不用 call 调用 JavaScript 会自动的传入一个 this。因为这个 this 在传入的时候我们看不见这就导致了 this 的值总是让人很难确定。举例说明：

```

function a() { console.log(this) };

a();  // this是什么

答案是window，这可是和上面说的不一样啊，

a() 转换成 a.call(undefined)，应该是undefined啊。为什么是window。

这是因为如果JavaScript在浏览器环境运行，如果call的第一个参数是undefined或者

null，那么this会变成全局对象window（如果是node.js环境中，那么就是global）。

如果使用严格模式，传入的undefined就是undefined不会默认指向window

```

![](/madao.github.io/database/images/articles/javascript/thisAndProto/image5.png)

那来看这种情况：

```

let obj = {

    name: 'allen',

    sayName: function() {

        console.log(this.name)

    }

}

obj.sayName(); // 'allen'

obj.sayName() 转换为 obj.sayName.call(obj)

```

问题来了，那我怎么知道在调用的时候，call 的第一个参数传什么。

用上面例子说明：

```

obj.sayName 目的是不是要操作sayName前面.的那个对象

(xxx.a() 中的xxx)，或者说你期望obj.sayName打印出来哪一个

name，例子中只有一个name，改写一下；

var name = '小红';

var xiaoming = {

    name: '小明',

    sayName: function() {

        console.log(this.name)

    },

    son: {

        name: '小小明'

    }

}

var xiaogang = {

    name: '小刚'

}

1. 我想要打印小明的名字

xiaoming.sayName.call(xiaoming)

2. 我想要打印小小明的名字

xiaoming.sayName.call(xiaoming.son)

3. 我想要打印小刚的名字

xiaoming.sayName.call(xiaogang)

4. 我想要打印小红的名字

xiaoming.sayName.call(undefined)

```

![](/madao.github.io/database/images/articles/javascript/thisAndProto/image6.png)

看两道题：

以下代码都运行在浏览器环境下

```

1.

function a() {

    console.log(this)

}

a中的this是什么

2.

var obj = {

    name: '小白',

    sayName: function () {

        console.log(this.name)

    }

}

var name = '小黑'

var sayName1 = obj.sayName

sayName1();

console.log 出来的name是什么

3.

var button = document.getElementsByClass('btn');

button.onclick = function() {

    console.log(this)

}

```

答案：

1. this 的值不确定。因为 a 并没有传入参数，或者说被调用，所以 this 的值不确定。
2. name 是 小黑 ，因为 sayName1() 没有对象.它，不知道它要操作哪个对象
   所以 call 传入 undefined，sayName1.call(undefined)，浏览器环境下又不是严格模式，那么 call
   传入的 undefined 被自动的指向了 window，全局用 var 声明一个变量，相当于
   给全局对象增加一个属性。所以 this.name === window.name
3. 因为我们无法确定 button.onclick 转换成 call 传入的参数，所以只能去看文档，
   ![](/madao.github.io/database/images/articles/javascript/thisAndProto/image7.png)
   根据文档确定 this 指的就是 button，但这只是一般的情况，
   如果`button.onclick.call({xxx: '', yyy: ''})`这样调用，
   那 this 就不是 button 了。

_知识点_：只有 var 在全局作用域声明的变量会变成全局对象的属性，let 和 const 不会。

![](/madao.github.io/database/images/articles/javascript/thisAndProto/image8.png)

构造函数也是一样的，如图：

![](/madao.github.io/database/images/articles/javascript/thisAndProto/image9.png)

基本上 this 的值我们可以通过 call 的第一个参数确定，但是 ES6 出现了一种新语法**箭头函数**

**箭头函数**又是另一种情况了

![](/madao.github.io/database/images/articles/javascript/thisAndProto/image10.png)

前面说 this 通过函数 call 的第一个参数确定，所以说 this 是一个函数的参数，箭头函数的 this 也可以用这个套路确定吗？

答案是：不行

例子：

```

var name = '阿花';

var foo = () => { console.log(this.name) };

foo.call({ name: '阿水' });  // '阿花'

```

![](/madao.github.io/database/images/articles/javascript/thisAndProto/image11.png)

我们用 call 方法调用 foo，并传入了一个对象作为 this，但是 foo 不要，它不要我们传入的 this。

那箭头函数的 this 如何确定呢。

箭头函数自己没有 this。箭头函数里的 this，是它定义的时候的“外面”的 this。

上面例子说明， `var foo = () => { console.log(this.name) };`foo 定义在全局作用域中，它外面的 this 是什么。

![](/madao.github.io/database/images/articles/javascript/thisAndProto/image12.png)

再来看几个例子：

```

var foo = {

    name: '张三',

    sayName: function() {

        var sayName1 = () => { console.log(this.name) }

        sayName1();

    }

}

foo.sayName();  // '张三'

sayName1外面是sayName函数。sayName函数的this是什么？

foo.sayName()  转换成 foo.sayName.call(foo)

所以sayName函数的this是 foo

```

```

var foo = {

    name: '张三',

    sayName: () => {

        console.log(this.name)

    }

}

var name = '李四'

foo.sayName(); // '李四'

sayName 外面是什么，是全局对象window，所以是李四

```

```

function foo() {

  setTimeout(() => { console.log(this.name) }, 1000)

}

var name = '王麻子'

foo.call({ name: '赵老爷' }); // '赵老爷'

() => { console.log(this.name) } 外面是foo，foo通过call传入一个对象作为this，

那么() => { console.log(this.name) }找到外面也就是foo的this就是{ name: '赵老爷' }

```

### 3. 构造函数

构造函数就是返回一个新对象的函数，它的出现是为了更优雅的创建对象。

前面说了原型链，那么还是用前面的例子举例，如何用函数批量的创建对象。

```

let humans = [];

let common = {

    heart: 1,

    eat: function(){},

    walk: function(){},

    sleep: function(){},

    laugh: function(){},

    cry: function(){}

}

function CreateHuman(id) {

    let human = {};

    human.id = id;

    human.__proto__ = common;

    return human;

}

for(let i = 0; i < 10; i++) {

    humans.push(CreateHuman(i))

}

```

这样我们就得到了 10 个人类对象。CreateHuman 就叫做构造函数

但是上面的代码有两个问题，

1. 不能显式的用`__proto__`

   原因：[`Object.prototype.__proto__`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/proto)

2. 代码结构太松散，common 对象和 CreateHuman 之间的关系不明确。

第二条还好说我们改成这样：

```

 let humans = [];

 function CreateHuman(id) {

     let human = {};

     human.id = id;

     human.__proto__ = CreateHuman.common;

     return human;

 }

 CreateHuman.common = {

     heart: 1,

     eat: function(){},

     walk: function(){},

     sleep: function(){},

     laugh: function(){},

     cry: function(){}

 }

 for(let i = 0; i < 10; i++) {

     humans.push(CreateHuman(i))

 }

```

common 这个属性在 JavaScript 中就被叫做 prototype。也就是这样

```

 let humans = [];

 function CreateHuman(id) {

     let human = {};

     human.id = id;

     human.__proto__ = CreateHuman.prototype;

     return human;

 }

 CreateHuman.prototype = {

     heart: 1,

     eat: function(){},

     walk: function(){},

     sleep: function(){},

     laugh: function(){},

     cry: function(){}

 }

 for(let i = 0; i < 10; i++) {

     humans.push(CreateHuman(i))

 }

```

第二条解决了，第一条怎么办？

用 new。

用了 new 之后我们的代码就变成这样

```

 let humans = [];

 function CreateHuman(id) {

     this.id = id;

 }

 CreateHuman.prototype = {

     heart: 1,

     eat: function(){},

     walk: function(){},

     sleep: function(){},

     laugh: function(){},

     cry: function(){}

 }

 for(let i = 0; i < 10; i++) {

     humans.push(new CreateHuman(i))

 }

 可以看到少了三行代码，分别是

 1. let human = {}; 不用自己创建对象，new 帮你创建这个对象，this = {}

 2. human.__proto__ = CreateHuman.prototype; 这一句也不需要了，new 帮你绑定原型链

 3. return human;  不用你return，new帮你return

```

这就是 new 的作用。

上面代码还有一个问题。那就是当你 new 一个构造函数的时候，这个构造函数的 prototype 属性就已经存在了，它里面会默认的有一条属性 constructor，这个属性用来记录这个构造函数创建的对象是由谁创建的。

如果按照上面这样写，还需要加一行代码

```

 let humans = [];

 function CreateHuman(id) {

     this.id = id;

 }

 CreateHuman.prototype = {

     constructor: CreateHuman,    // 新增

     heart: 1,

     eat: function(){},

     walk: function(){},

     sleep: function(){},

     laugh: function(){},

     cry: function(){}

 }

 for(let i = 0; i < 10; i++) {

     humans.push(new CreateHuman(i))

 }

 当然，你也可以这样写

 CreateHuman.prototype.eat = function(){}

 ...

 CreateHuman.prototype.cry = function(){}

 这样就不用重新指定constructor的指向了

```

构造函数有几个注意点：

1. 构造函数首字母一般使用大写
2. 如果构造函数没有参数可以省略()
3. 命名可以不用 create，因为它就是用来创建对象的

上面代码的最终版本就是这样：

```

let humans = [];

function Human(id) {

    this.id = id;

}

Human.prototype = {

    constructor: Human,

    heart: 1,

    eat: function(){},

    walk: function(){},

    sleep: function(){},

    laugh: function(){},

    cry: function(){}

}

for(let i = 0; i < 10; i++) {

    humans.push(new Human(i))

}

```

### 4. 继承

终于说到继承了，我看高程的时候，我的天啊。好多种继承方式啊，什么寄生式，组合寄生式。不是说它讲的不好，主要是太多了，我记不住。。。

```

如何让函数Jack拥有函数Human的自有属性和共有属性?

function Human(config) {

    this.complexion = config.complexion

    this.gender = config.gender

}

Human.prototype = {

    constructor: Human,

    eat: function(){},

    sleep: function(){},

    run: function(){},

    say: function(){}

}

function Jack(config) {

}

思路：

共有属性好说，就是让 Jack.prototype.__proto__ = Human.prototype就行了，

自有属性怎么办，我们看Human中的代码是this.xxx = xxx，那么在函数Jack中

把this传进去执行一下Human是不是就可以了。

实现：

function Jack(config) {

    Human.call(this, config)

    this.name = config.name

    this.city = config.city

    this.height = config.height

}

Jack.prototype = {

    constructor: Jack,

    writeCode: function(){},

    playGames: function(){}

}

Jack.prototype.__proto__ = Human.prototype

问题：

   __proto__ 不能用

解决问题思路：

  由于 __proto__ 不能用，所以用new解决，new一下Human我们不就可以实现

  Jack.prototype.__proto__ = Human.prototype这句代码了吗

第一次解决：

  function Jack(config) {

    Human.call(this, config)

    this.name = config.name

    this.city = config.city

    this.height = config.height

  }

  Jack.prototype = new Human({})

  Jack.prototype.constructor = Jack

  Jack.prototype.writeCode = function(){}

  Jack.prototype.playGames = function(){}

  要先new Human才行，不然writeCode这些Jack共有的方法就没了。

  结果如下：

```

![](/madao.github.io/database/images/articles/javascript/thisAndProto/image13.png)

```

新的问题：

  又出现重复的属性了。

解决思路：

  如果有一个空函数，让这个空函数.prototype = Human.prototype，然后我们再

  Jack.prototype = new 这个空函数，不就可以了吗？

第二次解决：

  function Jack(config) {

    Human.call(this, config)

    this.name = config.name

    this.city = config.city

    this.height = config.height

  }

  function fakeFn(){}

  fakeFn.prototype = Human.prototype

  Jack.prototype = new fakeFn()

  Jack.prototype.constructor = Jack

  Jack.prototype.writeCode = function(){}

  Jack.prototype.playGames = function(){}

  结果如下

```

![](/madao.github.io/database/images/articles/javascript/thisAndProto/image14.png)

这下就完美解决了，这就是 JavaScript 中实现继承的过程。

到了 ES5，JavaScript 新增了一个方法 Object.create。有了这个方法，上面例子中的代码就可以写成这样

```

 function Human(config) {

    this.complexion = config.complexion

    this.gender = config.gender

 }

 Human.prototype = {

    constructor: Human,

    eat: function(){},

    sleep: function(){},

    run: function(){},

    say: function(){}

 }

 function Jack(config) {

    Human.call(this, config)

    this.name = config.name

    this.city = config.city

    this.height = config.height

  }

  Jack.prototype = Object.create(Human.prototype)

  Jack.prototype.constructor = Jack

  Jack.prototype.writeCode = function(){}

  Jack.prototype.playGames = function(){}

  Object.create代替了之前的这三句代码

  function fakeFn(){}

  fakeFn.prototype = Human.prototype

  Jack.prototype = new fakeFn()

  所以在不支持Object.create环境中，使用

  function fakeFn(){}

  fakeFn.prototype = Human.prototype

  Jack.prototype = new fakeFn()

  这三句代码进行Object.create的兼容就行了

```

到了 ES6，JavaScript 又双叒叕出了新语法 class

那么上面的继承方法用 class 怎么写

```

class Human {

    constructor(config) {

        this.complexion = config.complexion

        this.gender = config.gender

    }

    eat(){}

    sleep(){}

    run(){}

    say(){}

}

class Jack extends Human {

    constructor(config) {

        super(config)

        this.name = config.name

        this.city = config.city

        this.height = config.height

    }

    writeCode(){}

    playGames(){}

}

```

画一张图对比下：

![](/madao.github.io/database/images/articles/javascript/thisAndProto/image15.png)
