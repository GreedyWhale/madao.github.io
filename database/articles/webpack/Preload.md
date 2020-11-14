上一篇总结了一下 webpack 的代码分割相关的知识，webpack 可以对同步引入的模块和异步引入的模块进行分割，将同步引入的第三方模块进行分割打包到一起，可以利用浏览器的缓存提高页面再次打开的速度，有助于提高页面首次打开的速度，当异步引入的模块是通过满足某些条件的时候再进行加载的时候，有助于提高页面首次打开的速度，就是 webpack 文档上说的懒加载。

### 一. 懒加载

还是使用上篇文章的配置，[GitHub](https://github.com/GreedyWhale/webpack_demo/tree/SplitChunksPlugin)。看一下懒加载的效果：

![](/madao.github.io/database/images/articles/webpack/preload/image.gif)

当用户点击了按钮，才会去加载 dayjs，然后输出对应的日期，代码如下：

```
function getCurrentDate () {
  return import(/* webpackChunkName: "dayjs" */ 'dayjs').then(({ default: dayjs }) => {
    const p = document.createElement('p')
    p.textContent = dayjs().format('YYYY年MM月DD日 HH:mm:ss')
    return p
  })
}

const button = document.createElement('button')
button.textContent = '点我'
document.body.appendChild(button)
button.onclick = () => {
  getCurrentDate().then(el => document.body.appendChild(el))
}
```

但是要注意，如果点击按钮后异步引入的模块过大的话，那么还是会造成交互卡顿。

这里介绍一个 chrome 的强大功能，查看页面的加载 js 的覆盖率。

还是使用上面这段代码，用浏览器打开打包后的 html 文件

![](/madao.github.io/database/images/articles/webpack/preload/image1.png)

点击 coverage 呼出对应的界面，然后点击录制按钮，刷新页面

![](/madao.github.io/database/images/articles/webpack/preload/image2.png)

这时候就可以看到，页面加载的 js 文件的代码覆盖率

![](/madao.github.io/database/images/articles/webpack/preload/image3.png)

可以看到，页面加载的这两个 js 文件呢，都只有一半的覆盖率，什么意思呢？就是页面在加载完成后，有一半的 js 代码是没有执行过的，用户加载了很多不必要的代码。

注意这里的百分比显示的是没有执行到的代码的百分比。

使用 webpack-bundle-analyzer 这个插件，看下打包后的代码都是些什么

![](/madao.github.io/database/images/articles/webpack/preload/image4.png)

vendors.js 都是一些 babel 的 polyfill，dayjs 就是 dayjs 的源码，main.js 就是业务逻辑。

- dayjs 是异步引入的所以页面加载不会去加载这个模块。
- vendors.js 主要是因为业务逻辑代码用到了新的语法才会有这些 polyfill

所以只要优化 main.js，那么就可以提高代码覆盖率了。

首先在 coverage 面板点开 main.js，这是可以看到，左侧有红色标注和和绿色标注的代码：

![](/madao.github.io/database/images/articles/webpack/preload/image5.png)

抛开 webpack 自身的一些逻辑，这段代码就是上面例子中的代码了，可以看到 onclick 里面的代码都是红色，那么这些代码就可以使用异步引入模块方式去加载它，提供覆盖率。

1. 新建 clickHandler.js

   ```
   function clickHandler () {
    return import(/* webpackChunkName: "dayjs" */ 'dayjs').then(({ default: dayjs }) => {
      const p = document.createElement('p')
      p.textContent = dayjs().format('YYYY年MM月DD日 HH:mm:ss')
      document.body.appendChild(p)
    })
   }

   export default clickHandler;
   ```

2. 修改 index.js

   ```
   const button = document.createElement('button')
   button.textContent = '点我'
   document.body.appendChild(button)
   button.onclick = () => {
     import( /* webpackChunkName: "clickHandler" */ './clickHandler')
       .then(({
         default: clickHandler
       }) => clickHandler())
   }
   ```

重新打包

![](/madao.github.io/database/images/articles/webpack/preload/image6.png)

重新打开页面

![](/madao.github.io/database/images/articles/webpack/preload/image7.png)

可以看到未使用率下降了，点击按钮也会加载对应的 js 代码。

虽然提升的不多，但是也是一种优化思路，通过这种查看代码覆盖率 + webpack 的代码分割功能可以针对性的做一些优化，到这里就能理解为什么 webpack 的 SplitChunksPlugin 这个插件也是默认只对 async 方式引入的模块进行分割，因为这样可以有效的提高页面打开的速度。

### 二. preload、prefetch

前面也说到，如果用户点击了一个按钮，再去加载相应的模块，模块代码过多时，交互就会卡顿，对于这种情况 webpack 页提供了解决办法，那就是：

- /_ webpackPrefetch: true _/
- /_ webpackPreload: true _/

使用方法：

```
import(
  /* webpackChunkName: "clickHandler" */
  /* webpackPreload: true */
  './clickHandler')
  .then(({
    default: clickHandler
  }) => clickHandler())

import(
  /* webpackChunkName: "clickHandler" */
  /* webpackPrefetch: true */
  './clickHandler')
  .then(({
    default: clickHandler
  }) => clickHandler())
```

当这样写了之后，异步引入的代码就不会等到用户触发了某些操作再去加载了，而是会提前加载这些资源

它们的区别就是 preload 会和父 chunk（在例子中父 chunk 就是 main.js）并行加载，prefetch 则会等到浏览器空闲时再去加载。

中文文档的区别截图过来

![](/madao.github.io/database/images/articles/webpack/preload/image8.png)

还有一个要注意的地方是 webpack 4.6.0+ 才支持这些，我自己用 4.35.2 版本测试，prefetch 是支持的，但是 preload 不支持。

[源码](https://github.com/GreedyWhale/webpack_demo/tree/prefetch_preload)
