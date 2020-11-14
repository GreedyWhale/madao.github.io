### 一. 什么是 Code Splitting

Code Splitting 顾名思义就是代码分割，为什么需要做代码分割呢？

按照之前的文章中的配置，webpack 在打包完成之后会将所有的 js 代码，打包成一个文件 main.js。前端的性能优化手段中中有一条就是减少 http 请求，但是减少 http 请求的同时，也要考虑请求资源的大小，当业务代码十分庞大的时候，将所有的 js 打包成一个文件这种方法就变得不可取了，举个简单的例子：

在日常开发中一般会用到第三方的库，一般情况下第三方库的源码我们是不会去修改的，但是业务逻辑代码是要根据需求进行修改的，如果将它们都打包到一起，那么一旦改了业务逻辑代码，用户重新访问页面的时候，就要连同第三方库的代码及业务代码全部重新下载一遍，但其实第三方库那一部分代码是没必要重新下载的，那么只要将不经常改动的代码打包到一起，经常改动的代码打包到一起，利用浏览器的缓存将这些不经常改动的代码缓存起来，就可以让用户在每次访问页面的时候减少请求的次数及请求资源的大小。

### 二. 使用 webpack 的 Code Splitting

[官方文档](https://webpack.js.org/guides/code-splitting/)

按照教程来：

1. 安装 lodash

   `yarn add lodash`

2. 新建 test1.js、test2.js

   test1.js

   ```
   import _ from 'lodash'

   const a = _.chunk([1,2,3,4], 2)
   console.log('test1.js')
   console.log(a)
   ```

   test2.js

   ```
   import _ from 'lodash'

   const b = _.chunk(['a', 'b', 'c', 'd'], 2)
   console.log('test2.js')
   console.log(b)
   ```

3. 在 index.js 中引入它们

   ```
   import './test1'
   import './test2'
   ```

   如果配置了 package.json 中的 sideEffects，那么记得要将它删除或者改成：

   ```
   "sideEffects": [ "./src/**/*.js"]
   ```

4. 打包看下结果

   代码是可以运行的

   ![](/madao.github.io/database/images/articles/webpack/code_splitting/image.png)

5. 将 lodash 拆分出来

   在 webpack.common.js 加上

   ```diff
   module.exports = {
   +   optimization: {
   +       splitChunks: {
   +           chunks: 'initial'
   +       }
   +   }
   }
   ```

   现在打包结果：

   ![](/madao.github.io/database/images/articles/webpack/code_splitting/image1.png)

   现在打包后就多出了一个文件，vendors~main.js，这个就是拆分出来的 lodash 源码。

   现在的文件名是 webpack 默认配置的名字，如果想要自定义文件名，可以这样配置

   在 webpack.common.js 中添加：

   ```diff
   module.exports = {
    output: {
      filename: '[name]_[hash].js',
   +   chunkFilename: '[name].js',
      path: path.resolve('dist')
    },
    optimization: {
      splitChunks: {
   +      chunks: 'initial'
   +      cacheGroups: {
   +        vendors: {
   +          filename: 'common.js',
   +          test: /[\\/]node_modules[\\/]/,
   +        }
        }
      }
    }
   }

   ```

   chunkFilename 就是设置非入口文件打包后的名称，意思就是按照目前的配置，entry 配置中只配置了 index.js 这个文件作为打包入口，那么 code splitting 把代码拆出来后的文件就是非入口文件，chunkFilename 就是设置这些文件的文件名。

   这样配置后分离后的文件名就是 common.js。

   那么如果你还想继续细分，比如有两个模块一个 lodash，一个 dayjs，想要把他们全部单独分离，不要全部放在 common.js，那么可以这样配置

   ```diff
   module.exports = {
     output: {
       filename: '[name]_[hash].js',
   +   chunkFilename: '[name].js',
       path: path.resolve('dist')
     },
     optimization: {
       splitChunks: {
   +      chunks: 'initial'
   +      minSize: 1000,
   +      cacheGroups: {
   +        vendors: {
   +          test: /[\\/]node_modules[\\/]/,
   +          name (module) {
   +            const moduleFileName = module.identifier().split('/').reduceRight(item => item)
   +            return moduleFileName.split('.')[0]
   +          }
   +        }
         }
       }
     }
   }

   ```

   test1.js

   ```diff
   import _ from 'lodash'
   + import dayjs from 'dayjs'

   const a = _.chunk([1,2,3,4], 2)
   console.log('test1.js')
   console.log(a)
   + console.log(dayjs().format('YYYY/MM/DD HH:mm:ss'))
   ```

   test2.js

   ```diff
   import _ from 'lodash'
   + import dayjs from 'dayjs'

   const b = _.chunk(['a', 'b', 'c', 'd'], 2)
   console.log('test2.js')
   console.log(b)
   + console.log(dayjs().format('YYYY年MM月DD日 HH:mm:ss'))
   ```

   注意上面的配置中添加了一个 minSize，这是因为 dayjs 这个库很小，webpack 默认配置中代码分离最小为 30kb，所以加上了这个条件，所以这个参数是要看实际情况的，现在打包的结果就是这样：

   ![](/madao.github.io/database/images/articles/webpack/code_splitting/image2.png)

   splitChunks 配置的默认值可以在文档中找到，[链接](https://webpack.js.org/plugins/split-chunks-plugin/#splitchunks-chunks)

   又是一堆的配置啊。。

### 三. 异步导入模块

前面的例子都是同步导入模块，还有一种导入模块的方法是使用 import 函数，这样可以动态的导入模块

import 函数，[详情](https://es6.ruanyifeng.com/#docs/module#import)

举个例子：

index.js

```diff
import './test1'
import './test2'

+ function getCurrentTime () {
+   return import('dayjs').then(({ default: dayjs }) => {
+     const p = document.createElement('p')
+     p.textContent = dayjs().format('YYYY年MM月DD日 HH:mm:ss')
+     return p
+   })
+ }

+ const button = document.createElement('button')
+ button.textContent = '点我'
+ document.body.appendChild(button)
+ button.onclick = () => {
+   getCurrentTime().then(el => document.body.appendChild(el))
+ }
```

**注意：** 要把 test1.js 和 test2.js 中的 dayjs 相关的代码注释，因为在 test1.js 和 test2.js 中 dayjs 是同步方式引入的，在 index.js 中又是异步引入的，我在自己测试的时候发现，如果入口文件只有一个，一个模块同时存在同步引入和异步引入，那么 weboack 就不会做代码分割了，也有可能是我的配置不对。

重新打包：

![](/madao.github.io/database/images/articles/webpack/code_splitting/image3.png)

现在多出了三个文件，array-includes.js 和 es.js 是 babel 的 polyfill，因为用到了新语法，所以会有这些文件，3.js 就是 dayjs 的源码，因为现在 dayjs 是异步导入的模块，按照文档上的例子，通过 import 这种方式导入的模块，webpack 会自动的进行代码分割，所以可以将 minSize 这个配置给删除掉，这样就就不会将 babel 的 polyfill 文件也给单独打包出来了。

删除 minSize 这个配置后重新打包：

![](/madao.github.io/database/images/articles/webpack/code_splitting/image4.png)

还是同样的问题，文件名不好认，那么 webpack 也提供了相应的配置，只要这样设置：

index.js

```
function getCurrentTime () {
  return import(/* webpackChunkName: "dayjs" */ 'dayjs').then(({ default: dayjs }) => {
    const p = document.createElement('p')
    p.textContent = dayjs().format('YYYY年MM月DD日 HH:mm:ss')
    return p
  })
}
```

在 import 函数中加一句/_ webpackChunkName: "dayjs" _/ ，这样打包后的文件就是这里设置的文件名。

异步导入模块有什么好处呢，现在将打包后的 html 文件打开：

![](/madao.github.io/database/images/articles/webpack/code_splitting/image5.png)

页面初始只会加载这些资源，当点击了按钮后：

![](/madao.github.io/database/images/articles/webpack/code_splitting/image6.png)

dayjs 才会被加载，这样做有助于提高页面第一次打开的速度。

[源码](https://github.com/GreedyWhale/webpack_demo/tree/SplitChunksPlugin)
