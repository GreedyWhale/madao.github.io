泛型是我之前没有听过的一个概念，看文档的介绍也是迷迷糊糊的，但是看了文档的例子基本是明白了，下面就总结下我对泛型的理解。

### 一. 泛型的概念

维基百科的解释：

> 泛型程序设计（generic programming）是程序设计语言的一种风格或范式。泛型允许程序员在强类型程序设计语言中编写代码时使用一些以后才指定的类型，在实例化时作为参数指明这些类型。

我自己的理解是，声明的函数时候不指定参数类型，返回值类型，而是在调用的时候根据传入的参数的类型再决定这些类型。例子：

假如有一个函数，返回值的类型和参数的类型一样，不用泛型要如何写：

```
function fn(params: any): any {
  return params;
}

const a = fn(123);
```

因为在声明函数的时候不能确定参数的类型，所以用了 any，既然返回值类型和参数类型相同，那么返回值的类型也是 any。

例子中的代码有一个问题，不能获取有用的 a 的类型，只能得到一个 any：

![](/caisr.github.io/database/images/articles/typescript/generic/image.png)

如果改成这样：

```
function fn<T>(params: T): T {
  return params;
}
```

这时候再看下 a 的类型：

![](/caisr.github.io/database/images/articles/typescript/generic/image1.png)

TypeScript 直接把 a 的值给你了。

用对象试下：

![](/caisr.github.io/database/images/articles/typescript/generic/image2.png)

这时候 TypeScript 会给你对象 key 的信息及 key 对应的值的类型。

用了泛型后可以获得很详细的函数返回值的类型。

### 二. 泛型的语法

基本的语法是：

```
function fn<T>(params: T): T {
  return params;
}
```

主要要理解 T 是什么东西？

其实 T 就是一个占位符，可以用其他的大小写字符来代替：

![](/caisr.github.io/database/images/articles/typescript/generic/image3.png)

调用的时候，也有两种方式：

```
function fn<T>(params: T): T {
  return params;
}

const a = fn({ name: "Allen", age: 19 });  // 第一种
const b = fn<string>('hi');  // 第二种
```

第一种方式是让 TypeScript 自己去推导 T 的类型，第二种是调用者告诉 TypeScript T 的类型。

在 TypeScript 中使用 Promise，就用到了泛型：

```
const p = new Promise(resolve => {
  resolve('hi')
})
```

![](/caisr.github.io/database/images/articles/typescript/generic/image4.png)

这样使用的话，TypeScript 并不能推导出 T 的类型。所以需要用<>来告诉 TypeScript 类型。

```
const p1 = new Promise<string>(resolve => {
  resolve('hi')
})
```

![](/caisr.github.io/database/images/articles/typescript/generic/image5.png)

### 三. 如何声明一个泛型类型

听起来挺绕的，意思就是假如有一个使用了泛型的函数，现在需要将这个函数赋值给一个变量，这个变量的类型如何声明。

有四种方式：

- 第一种

```
function fn<T>(params: T): T {
  return params;
}

const a: <T>(params: T) => T = fn;
```

- 第二种：

```
function fn<T>(params: T): T {
  return params;
}

const a: {<T>(params: T): T} = fn;
```

- 第三种

```
interface FnInterface {
  <T>(params: T): T
}

function fn<T>(params: T): T {
  return params;
}

const a: FnInterface = fn;
```

- 第四种

```
interface FnInterface<T> {
  (params: T): T
}

function fn<T>(params: T): T {
  return params;
}

const a: FnInterface<string> = fn;
```

这几种方式只能死记硬背了，需要注意的是第四种要明确的告诉 TypeScript 类型。

还有一点需要注意的是，在使用了泛型的函数中使用类，js 是这样的：

```
function fn(c) {
  return new c();
}
```

转换成 TypeScript 是这样：

```
function fn<T>(c: { new(): T; }): T {
  return new c();
}
```

或者：

```
function fn<T>(c: new() => T): T {
  return new c();
}
```

### 四. 泛型约束

泛型约束指的是，虽然在函数的参数类型，返回类型是在调用的时候确定的，但是可以在声明函数的时候，指定参数必须要包含哪些属性，例子：

```
function fn<T>(params: T): T {
  /*
  * Property 'length' does not exist on type 'T'
  * 这里会报错，因为不确定params的类型，所以params有可能不含有length属性，
  */
  console.log(params.length);
  return params;
}
```

改成这样：

```
interface ParamsInterface {
  length: number;
}

function fn<T extends ParamsInterface>(params: T): T {
  console.log(params.length);
  return params;
}
```

这就是泛型约束。
