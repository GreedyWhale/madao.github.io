TypeScript 是 JavaScript 的超集，所以 JavaScript 中的值的类型 TypeScript 都有，除了 JavaScript 中的 7 种类型，TypeScript 还额外的提供了 tuple，any，void，never，enum，在 TypeScript 中基础类型有：

- boolean
- number
- string
- 数组
- 元组
- enum
- any
- null
- undefined
- nerve
- object
- void
- symbol

JavaScript 中有的那 7 种就不说了，主要说下 TypeScript 额外提供的这 5 中类型。

### 一. 元组 tuple

我第一次知道元组的概念是学习 python 的时候知道的，我理解的元组就是不可变的数组，在 TypeScript 中不可变指的是元组长度和元组中每一项的值的类型不可变。例子：

```
let tuple: [number, string, boolean]

tuple = [1, 'hi', true]
tuple = ['hi', 123, false]  // 类型错误
tuple = [1, 'hi', true, 1]  // 长度错误
```

### 二. 枚举 enum

枚举是我从来没有用过的数据类型，直接看例子：

```
enum Days { Sun, Mon, tue, Wed, thu, Fri, Sat };

console.log(Days.Sun)  // 0
console.log(Days.Sat) // 6
```

![](/madao.github.io/database/images/articles/typescript/type/image.png)

文档的说明：

> 使用枚举类型可以为一组数值赋予友好的名字。

它默认的值是序号，从 0 开始，也可以给这些“名字”一些值：

![](/madao.github.io/database/images/articles/typescript/type/image1.png)

给其中一项一个数字，它后面的每一项就会根据这个数字递增，

如果给的不是数字类型，那么剩下的每一项都要手动赋值：

```
enum Days {
  Sun = "a",
  Mon = "b",
  tue = "c",
  Wed = "d",
  thu = "e",
  Fri = "f",
  Sat = "g"
}

console.log(Days.Sun); // a
console.log(Days.Sat); // g
```

除了通过名字获取值，也能通过值获取名字：

```
enum Days { Sun, Mon, tue, Wed, thu, Fri, Sat };
const name: string = Days[4];
console.log(name); // thu
```

注意：如果值是字符串就不能通过值获取名字。

### 三. any

any 就是任何类型，如果开发者不能确定值的类型，那么可以使用 any，虽然 any 很好用，但是不建议滥用，全部都写成 any 的话，还不如直接写 JavaScript，推荐一个 TypeScript 的配置项：

```
"compilerOptions": {
    "noImplicitAny": true
}
```

它的意思就是对隐含 any 类型的表达式和声明发出警告。举个例子：

```
function sayHi(name): void {
  console.log(`Hi! ${name}`)
}
sayHi('joey')
```

![](/madao.github.io/database/images/articles/typescript/type/image2.png)

### 四. void

void 表示没有任何类型，比如一个函数执行完没有 return 任何东西，那么就可以使用 void，变量也是可以声明成 void 类型的，一旦声明成 void 类型，那么变量只能被赋值成 undefined 或 null

![](/madao.github.io/database/images/articles/typescript/type/image3.png)

void 一般用于表示函数没有返回任何值

### 五. never

never 就更奇怪了，文档的解释：

> never 类型表示的是那些永不存在的值的类型。

它的应用场景有：

1. 函数根本不会有返回值，比如：

   ```
   function infiniteLoop(): never {
       while (true) {
           console.log(1)
       }
   }
   ```

   这个函数执行不完，所以根本就不会有返回值，这时候就可以使用 never

2. 总是会抛出错误的函数，比如：

   ```
   function error(message: string): never {
       throw new Error(message);
   }
   ```

never 看起来和 void 挺像，它们的区别是：

1. 函数没有任何返回值：void，函数不可能有返回值：never
2. never 类型是任何类型的子类型，可以赋值给任何类型，例子：
   ```
   let a: never;
   let b: void;
   let c: string = a;
   let d: number = b;  // Type 'void' is not assignable to type 'number'.
   ```

### 六. 类型断言

类型断言的意思是告诉 TypeScript 我知道这个值的类型，按照我断言的类型进行检查，比如:

```
function fn(params: string | number): string {
  if (params.length) {
    return params
  }
  return params.toString();
}
```

如果这样写 TypeScript 会报错：

![](/madao.github.io/database/images/articles/typescript/type/image4.png)

这时候就可以用类型断言：

```diff
function fn(params: string | number): string {
+   if ((params as string).length) {
+     return (params as string)
    }
    return params.toString();
}
```

```diff
function fn(params: string | number): string {
+ if ((<string>params).length) {
+   return (<string>params);
  }
  return params.toString();
}
```

类型断言的写法有两种：

```
(<类型>值)

(值 as 类型)
```
