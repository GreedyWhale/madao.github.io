最近在用 vue + typescript 写一个项目，由于是第一次使用 typescript，所以踩了一些坑，这里记录一下。

### 一. vue-property-decorator

使用 vue-cli 搭建项目的话，可以从它给出的 Home.vue 文件中，可以看到用 typescript 写单 vue 文件会有一些不同：

```
import { Component, Vue } from "vue-property-decorator";
import HelloWorld from "@/components/HelloWorld.vue"; // @ is an alias to /src

@Component({
  components: {
    HelloWorld
  }
})
export default class Home extends Vue {}
```

- vue-property-decorator

  从示例中看出 Component, Vue 这些是从 vue-property-decorator 这个库引入的，vue-property-decorator 是个什么呢？这是它的 GitHub 地址[vue-property-decorator](https://github.com/kaorun343/vue-property-decorator#Model)。

  从它的介绍来看，它说这个库是完全依赖另一个库[vue-class-component](https://github.com/vuejs/vue-class-component)推荐先去阅读这个库的 README，好吧，那只能先去看看 vue-class-component 这个库是什么了。

  在[vue 的官网](https://cn.vuejs.org/v2/guide/typescript.html#ad)中有提到：

  ![](/madao.github.io/database/images/articles/vue/vue_and_ts_one/image.png)

  我看了一些其他人写的文章，简单的说就是用 typescript 写 vue 每次都需要写一些额外形式的代码，vue-class-component 通过装饰器来减少这些重复的代码，vue-property-decorator 则是在 vue-class-component 的基础上增加了一些装饰器。

  - 装饰器

    在写 JavaScript 的时候，基本是没有接触过装饰器，还好之前学习 Python 的时候，接触到了这方面的知识，简单介绍下：

    ```
    #!/usr/bin/env python3
    # -*- coding: utf-8 -*-

    '''
    需求：想要给函数add和函数square增加执行时间打印功能
    '''

    def add(a, b):
        return a + b

    def square(a):
        return a ** 2
    ```

    方法一：直接在 add 和 square 执行前和执行后把时间打印出来即可：

    ```
    import time

    t1 = time.time()
    a = add(10, 20)
    print('add执行时间：%f' % (time.time() - t1))

    t2 = time.time()
    b = square(10)
    print('square执行时间：%f' % (time.time() - t2))
    ```

    方法二：将重复的代码封装起来

    ```
    def print_execute_time(func):
    def wrap(*args, **kwargs):
        t1 = time.time()
        result = func(*args, **kwargs)
        print('%s执行时间：%f' % (func.__name__, time.time() - t1))
        return result
    return wrap

    print_execute_time(add)(10, 20)
    print_execute_time(square)(10)
    ```

    就是把需要执行的函数当做参数，给需要执行的函数包一层。

    方法三：使用装饰者模式，在 Python 中是`@装饰函数名`语法：

    ```
    #!/usr/bin/env python3
    # -*- coding: utf-8 -*-

    import time

    def print_execute_time(func):
        def wrap(*args, **kwargs):
            t1 = time.time()
            result = func(*args, **kwargs)
            print('%s执行时间：%f' % (func.__name__, time.time() - t1))
            return result
        return wrap

    @print_execute_time  # 新增
        def add(a, b):
        return a + b

    @print_execute_time # 新增
    def square(a):
        return a ** 2
    add(10, 20)
    square(10)
    ```

    总结下就是，在不改写函数体或者函数调用方式的情况下，给函数增加一些新功能。虽然这是 Python 的例子，但是概念还是差不多的，JavaScript 中呢好像还处于提案阶段，具体参考阮一峰大神的[修饰器](http://es6.ruanyifeng.com/?search=super&x=0&y=0#docs/decorator)。

  既然用了 vue-property-decorator，那么 vue 的写法也就出现了一些不同，比如：

  ```
  import HelloWorld from "@/components/HelloWorld.vue"; // @ is an alias to /src

  @Component({
      components: {
          HelloWorld
      }
  })
  export default class Home extends Vue {}
      // data
      msg = 'hello'
      name = 'home'

      // props
      @Prop({ default: "text" })
      type!: string;
      @Prop()
      placeholder!: string;

      // methods
      say_name(): void {
          console.log(this.name)
      }
  ```

  好在是 vue-property-decorator 这个库给了不同装饰器的用法示例，可以去 GitHub 上参考。

### 二. `@Prop() private msg!: string;`

在看 vue-cli 给出的示例中，HelloWorld 这个组件中，prop 的写法是这样，发现一个特殊的地方，就是 msg 后面加了一个!，在 typescript 中这种写法的意思就是!前面的这个变量一定不是 undefined 或者 null，!叫非空断言操作符。[文档](https://www.tslang.cn/docs/release-notes/typescript-2.0.html)

### 三. 当函数的参数是对象的时候

es6 中加入了解构赋值这一新语法，所以我在写函数的时候喜欢这样写：

```
    const obj = {
        name: 'allen',
        age: 18
    }

    function say_something({ name, age }) {
        console.log(name, age)
    }
```

因为 typescript 中有静态数据类型这个东西，就是在声明变量的时候，就要定好该变量的类型，那么在函数中参数也是需要给它定好一个类型的：

```
    // 一开始我以为只要写Object就好了, 但是这样会报错

    const obj: Object = {
        name: 'allen',
        age: 18
    }

    function say_something({ name, age }: Object):void {
        console.log(name, age)
    }

    // 后来网上查了一下，需要写成这样：

    interface obj {
        name: string;
        age: number;
    }
    function say_something({ name, age }: obj):void {
        console.log(name, age)
    }

    // 或是这样：

    function say_something({ name, age }: any):void {
        console.log(name, age)
    }
```

### 四. `Element implicitly has an 'any' type because type 'Set<string>' has no index signature`

这是一个报错提示，出现在了我这样写的时候：

```
    export default class Test extends Vue {
        set_something(name: string, value: any):void {
            this[name] = value
        }
    }
```

要解决这个问题，需要这样写：

```
    export default class Test extends Vue {
        set_something(name: string, value: any):void {
            interface IParams {
                [key: string]: any
            }
            (<IParams>this)[name] = value
        }
    }
```

这就是目前碰到的一些小坑吧，在写项目的过程中参考了以下文章：

- [从 JavaScript 到 TypeScript ](https://tasaid.com/Blog/20171011231943.html)
- [ TypeScript 踩坑（持续更新）](https://segmentfault.com/a/1190000016927260#articleHeader3)
- [typescript 中函数参数为对象的时候怎么写](https://segmentfault.com/q/1010000010551872)

目前还未体会到 typescript 的强大之处，感觉就是写的很繁琐。/(ㄒ o ㄒ)/~~
