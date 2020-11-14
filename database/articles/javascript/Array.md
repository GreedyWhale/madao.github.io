### 1. 多维数组变为一维数组：

```
let arr = [1,[2, [3], [4, 5, 6], [7, 8, 9], 10, 11], 12, 13, 14, [15, 16, 17]]
```

- 方法一：递归

  ```
  function recursive(arr, newArr) {
      arr.forEach(item => {
          Array.isArray(item) ? recursive(item, newArr) : newArr.push(item)
      })
  }

  let newArr = [];
  recursive(arr, newArr)
  ```

  结果：

  ![](/madao.github.io/database/images/articles/javascript/array/image.png)

- 方法二：reduce

  ```
  let flattenDeep = (arr) => arr.reduce((start, current) => (
      Array.isArray(current) ?
      start.concat(...flattenDeep(current)) :
      start.concat(current)
  ), [])

  let newArr2 = flattenDeep(arr);
  ```

  结果：

  ![](/madao.github.io/database/images/articles/javascript/array/image1.png)

- 方法三：flat

  ```
  let newArr3 = arr.flat(Infinity);
  ```

  ![](/madao.github.io/database/images/articles/javascript/array/image2.png)

### 2. 二维数组变一维数组

```
let arr = [[1,2,3], [4,5,6], 7,8,9];
```

- 方法一：reduce

  ```
  let newArr = arr.reduce((start, current) => (start.concat(current)), [])
  ```

- 方法二：concat

  ```
  let newArr1 = [].concat(...arr)
  ```

- 方法三：concat + apply

  ```
  let newArr2 = [].concat.apply([], arr)
  ```

  结果：

  ![](/madao.github.io/database/images/articles/javascript/array/image3.png)

### 3. 将一维数组变为二维数组

```
let arr = [1,2,3,4,5,6,7,8,9,0];
```

- 方法：

  ```
  let group = (arr, length) => {
      let index = 0;
      let newArr = [];
      while (index < arr.length) {
          newArr.push(arr.slice(index, index += length))
      }
      return newArr;
  }
  ```

  结果：

  ![](/madao.github.io/database/images/articles/javascript/array/image4.png)

### 4. 关于 ruduce 的理解

reduce 是一个功能很强大的函数，由于平时不怎么使用所以这里写一下作为备忘。

- 语法

  arr.reduce(callback[, initialValue])

  - callback 有四个参数

    - Accumulator (acc) (累计器)
    - Current Value (cur) (当前值)
    - Current Index (idx) (当前索引)
    - Source Array (src) (源数组)

  - initialValue，作为第一次调用 callback 函数时的第一个参数的值。

- 用法

  reduce 会为数组中的每一个元素都执行一边 callback，如果传入了 initialValue 那么第一次执行 callback 中的 Accumulator 就是 initialValue 的值，callback 中的 Current Index 就是 0，如果不传,第一次执行 callback 中的 Accumulator 就是数组中的第一个元素，Current Index 就是 1

  例子：

  ```
  // 算出1+2+3+...+100的值
  let source = Array.from({length: 100}, (v, i) => i+1);
  let result = source.reduce((acc, cur) => acc + cur);
  ```

  最开始这种我都是用递归去算，使用 reduce 就显得很简单了。简单的说下计算过程

  ```
  let arr = [1,2,3,4]
  let res = arr.reduce((acc, cur) => acc + cur);

  // reduce的计算过程就像下面这样
  ((1+2)+3)+4

  /*
    就是第一次callback的执行结果，会作为第二次callback的acc参数传入。
    如果传入了initialValue值那么计算过程就是这样
  */

  let res1 = arr.reduce((acc, cur) => acc + cur, 100);

  (((100+1)+2)+3)+4
  ```
