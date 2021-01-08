- ### 类型

  JavaScript 中数据的类型有以下几种：

  - string
  - number
  - boolean
  - object
  - null
  - undefined
  - symbol

- ### 声明

  声明对象的方法有：

  1.  直接声明

      ```

      let obj = {

        key: value

      }

      ```

  2.  使用构造函数

      ```

      let obj = new Object();

      obj.key = value

      ```

      这两种方法创建出来的对象是一样的。

- ### 引用类型

  对象类型又被叫做引用类型，引用类型是什么意思呢？就是引用类型储存的是值的地址（也可以理解为指针）。

  1. 基本类型有

     - string
     - number
     - null
     - undefined
     - boolean
     - symbol

  2. 引用类型有

     - object

  它们的区别看下面例子

  ```

  let a = 1;

  let b = a;

  b = 2;

  console.log(a, b);   // 1  2

  let foo = {

    name: 'allen'

  }

  let bar = foo;

  bar.age = 17;

  console.log(foo, bar)// {name: "allen", age: 17} {name: "allen", age: 17}

  ```

  如果是基本类型，将它赋值给其他变量。赋值后的值和原始值彼此是独立的，改变其中一个另一个不会跟着改变，而引用类型不同，它储存的仅仅是一个地址，赋值给其他变量后，只是将地址赋值给了另一个变量，改变其中一个会导致另一个也跟着改变，就好像你将家里的钥匙给了其他的人，那个人给你家里放了一些东西，你回到家当然也能看到多出来的东西，你们两只是拿的钥匙相同。

  上面例子中对象的这种赋值就叫做浅拷贝，如果不想让赋值后的对象影响原始对象，那么就要用深拷贝的方法进行赋值。

  `JSON.parse(JSON.stringify(obj))`
  ![image.png](/madao.github.io/database/images/articles/javascript/object/image.png)

- ### 对象的属性

  对象的属性可以通过 `.`或者`[xxx]`这两种方式访问，在对象中属性名永远都是字符串，如果用了字符串以外的值作为对象的属性名，那么这个值会先转换为字符串，例子：

  ```

  let bar = {};

  let foo = {};

  bar[1] = 'd';

  bar[true] = 'e';

  bar[foo] = 's'

  bar['1'];  // 'd'

  bar['true']; // 'e'

  bar['[object Object]']; // 's'

  ```

- ### 属性描述符

  对象属性的描述符有：

  - value (属性的值)
  - writable (可写)
  - enumerable (可枚举)
  - configurable (可配置)

  可以使用 Object.getOwnPropertyDescriptor 获取对象的某个属性的描述符，也可以用 Object.getOwnPropertyDescriptors 获取对象的所有属性的描述符，例子：

  ![](/madao.github.io/database/images/articles/javascript/object/image1.png)

  这些属性描述符可以通过 Object.defineProperty 或 Object.defineProperties 进行修改设置，这两个方法的区别就和上面例子一样，一个是单个配置，一个是批量配置。

  - writable，控制对象的属性是否可以修改，例子：

    ```

    let foo = {};

    Object.defineProperty(foo, 'a', {

      enumerable: false,

      configurable: false,

      writable: false,

      value: 1

    });

    foo.a = 2;

    console.log(foo.a)  // 1

    ```

    在非严格模式下，修改会失败，但是不会报错，严格模式下会报错

  - configurable, 控制对象的属性是否可以用 defineProperty 进行配置，需要注意的是，一旦你通过 defineProperty 方法将某个属性的 configurable 设置为 false，那么这个属性就再也不能用 defineProperty 进行配置了，有个例外就是在 configurable 设置 为 false 的情况下，仍然可以将属性的 writable 从 true 改为 false，改为 false 之后就不能再改成 true 了，configurable 设置为 false 的状态下，会禁止使用 delete 删除这条属性。例子：

    ```

    let bar = {};

    Object.defineProperty(bar, 'a', {

      enumerable: false,

      configurable: false,

      writable: true,

      value: 1

    });  // 成功

    Object.defineProperty(bar, 'a', {

      enumerable: false,

      configurable: true,

      writable: true,

      value: 1

    });   // 报错 Uncaught TypeError: Cannot redefine property: a

    Object.defineProperty(bar, 'a', {

      enumerable: false,

      configurable: false,

      writable: false,

      value: 1

    });   // 成功

    Object.defineProperty(bar, 'a', {

      enumerable: false,

      configurable: false,

      writable: true,

      value: 1

    });  // 报错 Uncaught TypeError: Cannot redefine property: a

    bar.a;  // 1

    delete bar.a;

    bar.a;  // 1

    ```

  - enumerable，控制对象的属性是否出现在对象属性的枚举中，设置为 false，可以正常访问到，但不会出现在枚举中，例子：

    ```

    let bar = {};

    Object.defineProperties(bar,{

      a: {

        enumerable: false,

        configurable: true,

        writable: true,

        value: 1

      },

      b: {

        enumerable: true,

        configurable: true,

        writable: true,

        value: 2

      },

      c: {

        enumerable: true,

        configurable: true,

        writable: true,

        value: 3

      },

    })

    bar.a // 1

    for(let key in bar ) {

        console.log(key)  // b, c

    }

    ```

- ### getter 和 setter

  直接举例子说明：

  ```

  let bar = {a: 1, b:2};

  bar.a;   // 这就是getter

  bar.a = 3;  // 这就是setter

  ```

  也就是说可以通过 getter 和 setter, 监听到对象属性的变化。例子：

  ```

  let bar = {

    get a() {

      console.log('访问了 bar 的 a属性')

      return this._a;

    },

    set a(value) {

      console.log('设置了 bar 的 a属性')

      this._a = value;

    }

  }

  bar.a;  // 访问了 bar 的 a属性

  bar.a = 2; // 设置了 bar 的 a属性

  ```

  上面例子中，我们把 a 的值存在了\_a 里面，如果不这样做，直接放在 a 中，会造成 set 或 get 函数一直执行。

  这里就可以得出一个结论，对象的属性不一定是包含值，也可能是具有 getter/setter 的访问描述符。（当你给一个属性定义 setter 或者 getter，或者两者都有时，这个属性会被定义为“访问描述符”。）

  小知识点：

  1. in 操作符检查的是某个属性名是否存在而不是某个值是否存在，例子：

     ```

     let foo = {a: 1};

     let bar = [5,6,7,8];

     ('a' in foo);  // true

     (5 in bar); // false

     (0 in bar); // true，因为bar中的属性名是0、1、2、3

     ```

  2. 书中（《你不知道的 JavaScript（上卷）》）有说到一个很有意思的知识点，那就是这个

     ```

     typeof null  // "object"

     null instanceof Object   // false

     ```

     书上说，JavaScript 中不同的对象在底层都表示为二进制，如果二进制前三位都是 0 的话，就会判断为 object 类型，null 的二进制全是 0，所以 typeof null 为'object'，而 null 本身就是 null 类型而不是 object 类型，所以 null instanceof Object 为 false

  3. 使用 `let obj1 = Object.create(null)` 创建的对象比 `let obj = {}`更空

     例子：
     ![image.png](/madao.github.io/database/images/articles/javascript/object/image2.png)
