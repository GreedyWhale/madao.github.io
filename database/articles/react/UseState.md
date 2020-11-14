### Hook 简介

官网说的最贴切，直接抄过来：

> Hook 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。

useState 是 React 内置的 hook，它的具体用法如下：

```
const [state, setState] = useState(initialState);
```

返回一个 state，以及更新 state 的函数。

在初始渲染期间，返回的状态 (state) 与传入的第一个参数 (initialState) 值相同。

setState 函数用于更新 state。它接收一个新的 state 值并将组件的一次重新渲染加入队列。

```
setState(newState);
```

在后续的重新渲染中，useState 返回的第一个值将始终是更新后最新的 state。

看下效果

![](/madao.github.io/database/images/articles/react/useState/code.png)

![](/madao.github.io/database/images/articles/react/useState/image.png)

### useState 原理

仔细观察上面给出的例子，每次点击按钮都会执行 setCount，每执行一次 setCount，控制台会在每次点击按钮的时候将当前的 count 值打印出来，意味着组件重新渲染了（App 重新执行了）。

问题出现了，App 每次执行

```
const [count, setCount] = useState(0)
```

这句代码也会被执行，那么 count 应该被初始化为 0 才对，但从结果上看，打印出来的 count 值是正确的，所以问题就是同样的函数，同样的参数但是每次执行的获得的结果却不同，诶！说好的函数式编程呢。

根据例子分析，现在知道的事情有以下几件：

1. setCount 会修改某个数据，并存起来，假设这个数据叫 x
2. setCount 会触发渲染
3. useState 会读取 x，获取最新的值返回

现在试着自己实现 useState

![](/madao.github.io/database/images/articles/react/useState/code1.png)

![](/madao.github.io/database/images/articles/react/useState/image1.png)

虽然写的很丑陋，但是确实实现了 useState 的效果 React 的实现肯定不是这样的，原理基本是一样

仔细看上面代码，这个代码有一个 bug，如果存在多个 state 的时候，就会出现冲突，接下来进行优化

![](/madao.github.io/database/images/articles/react/useState/code2.png)

将 state 的数据结构改为

```
{ value: [], index: 0 }
```

这种格式，value 是一个数组，用于储存多个 state，index 则是为了记录下一次设置 state 的时候的下标，每次执行 setState 的时候将 index 初始化为 0，这样组件重新渲染后获取的 state 顺序就不会乱了。现在测试一下

![](/madao.github.io/database/images/articles/react/useState/code3.png)

![](/madao.github.io/database/images/articles/react/useState/image2.png)

从这个实现就能看出，useState 的调用顺序是很重要的，所以在 React 中是不能在 if 中写 useState 的：

![](/madao.github.io/database/images/articles/react/useState/code4.png)

![](/madao.github.io/database/images/articles/react/useState/image3.png)

### 对 useState 的误解

看个例子：

![](/madao.github.io/database/images/articles/react/useState/code5.png)

先 +1，后 log
![](/madao.github.io/database/images/articles/react/useState/image4.png)
先 log，后 +1
![](/madao.github.io/database/images/articles/react/useState/image5.png)

先 +1，后 log 结果符合预期，但是先 log，后 +1，log 出的 n 是旧的 n

这是因为 setN 并不会改变 n，它会触发重新渲染，重新渲染之后的 n 和之前的 n 是两个不同的 n，先 log 再加 1 的时候，n 还是 0，调用 setTimeout，2 秒后打印 n，然后 +1 触发重新渲染获得新的 n，由于 setTimeout 是在重新渲染获之前就调用了，所以它根据作用域找到的 n 就是重新渲染之前的 n，就是 0

所以要记住 setState 并不会修改 state
