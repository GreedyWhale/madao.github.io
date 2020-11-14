接口：用来描述对象拥有什么属性或方法，TypeScript 接口用 interface 来定义，例子：

```
interface Student {
    name: string;
}
```

### 一. 接口的语法

现在有一个对象如下：

```
const allen = {
  name: 'allen',
  number: 1,
  class: '一班'
}
```

用接口描述如下：

```
interface Student {
  name: string;
  number: number;
  class: string;
}

const allen: Student = {
  name: 'allen',
  number: 1,
  class: '一班'
}
```

使用上面的接口规定对象 allen 的类型后，对象 allen 的属性就必须有接口中定义的那些属性，不能多也不能少，而且属性的值的类型也必须符合接口定义的类型。

#### 可选属性

可选属性指的是对象中可能有的属性，语法：

```
interface MyInterface {
    key?: type;
}
```

用上面的例子来说明：

```diff
interface Student {
  name: string;
  number: number;
  class: string;
+ nickname?: string;  // 可选属性
}

const allen: Student = {
  name: 'allen',
  number: 1,
  class: '一班'
}

const jeoy: Student = {
  name: 'joey',
  number: 2,
  class: '二班',
  nickname: 'muggle'
}
```

接口 Student 中有一个可选属性昵称，有的学生没有有的学生没有，只要在接口的属性后面加一个`?`，那么这个属性就是可选属性。

#### 只读属性

顾名思义只读属性就是不能再次修改的属性，语法：

```
interface MyInterface {
    readonly key?: type;
}
```

例子：

```diff
interface Student {
+ readonly name: string;
  number: number;
  class: string;
  nickname?: string;
}

const allen: Student = {
  name: "allen",
  number: 1,
  class: "一班"
};

allen.name = "jack";  // Cannot assign to 'name' because it is a read-only property.
```

假如你不知道一个对象的所有属性，只知道一部分，那么可以这样写：

```
interface Student {
  readonly name: string;
  number: number;
  class: string;
  nickname?: string;
  [key: string]: any;
}
```

或者更宽松一点：

```
interface Student {
  [key: string]: any;
}
```

这样写的意思是对象拥有任意数量的属性，属性的类型是字符串，属性的值的类型是任何类型。

### 二. 用接口描述函数

用接口函数的语法如下：

```
interface MyFn {
    (param1: type, param2: type): type;
}
```

`(param1: type, param2: type)` 这一部分是函数参数的类型，后面的是函数返回值的类型。例子：

```
interface Calculator {
  (a: number, b: number): number;
}
let add = <Calculator>function add (a, b) {
  return a + b;
}
let minus: Calculator = function(a, b) {
  return a - b;
}
```

这两种写法都是可以的。

### 三. 用接口描述数组

用接口描述数组也很简单，语法：

```
interface NameList {
  [index: number]: string;
}

let nameList = ['allen', 'tom', 'jack'];
```

### 四. 用接口描述类

接口描述类需要用 implements，例子：

```
interface CarInterface {
  name: string;
  speed: number;
  speedUp(speed: number): number;  // 类的方法
  speedDown(speed: number): number; // 类的方法
}

class Car implements CarInterface {
  constructor(public name: string) {} // 这里的public和this.name = name的效果是一样的
  speed = 0;
  speedUp(speed: number): number {
    this.speed += speed;
    return this.speed;
  }
  speedDown(speed: number): number {
    if (this.speed) {
      this.speed -= speed;
    }
    return this.speed;
  }
}

const porsche = new Car("Porsche");
porsche.speedUp(100);
porsche.speedDown(10);
console.log(porsche.speed); // 90

```

### 五. 接口的继承

接口继承使用 extends 关键字，例子：

```
interface Human {
  head: number;
  body: number;
  foot: number;
  gender: string;
}

interface Student extends Human {
  readonly name: string;
  number: number;
  class: string;
  nickname?: string;
}
```

接口可以多层继承，有两种写法

- 第一种

```
interface A {
  a: string;
}

interface B extends A {
  b: string;
}

interface C extends B {
  c: string;
}

const obj: C = {
  a: 'a',
  b: 'b',
  c: 'c'
}
```

- 第二种

```
interface A {
  a: string;
}

interface B {
  b: string;
}

interface C extends A, B {
  c: string;
}

const obj: C = {
  a: 'a',
  b: 'b',
  c: 'c'
}
```

### 六. 混合类型

混合类型就是一个对象同时具有多种类型，比如一个函数对象，同时它又拥有一个属性，这个属性也是函数，例子：

```
function sayName(name) {
  console.log(name);
}
sayName.sayAge = (age) => {
  console.log(age);
}
```

用接口描述这个函数：

```
interface SayNameInterface {
  (name: string): void;
  sayAge(age: number): void;
}
```

同时需要改变一下 sayName 的声明方式：

```
interface SayNameInterface {
  (name: string): void;
  sayAge(age: number): void;
}

let sayName: SayNameInterface = (function(): SayNameInterface {
  const sayName = <SayNameInterface>function(name) {
    console.log(name);
  };
  sayName.sayAge = age => {
    console.log(age);
  };
  return sayName;
})();

sayName('allen'); // 'allen'
sayName.sayAge(123); // 123

```

如果觉得上面写法复杂，这样写也是可以的：

```
const sayName = <SayNameInterface>function(name) {
  console.log(name);
};
sayName.sayAge = age => {
  console.log(age);
};
```

利用类型断言实现。
