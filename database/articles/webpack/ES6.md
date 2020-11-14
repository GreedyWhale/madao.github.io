接着上一篇文章中的配置

### 一. Webpack Dev Server

在实际的开发中，不会每次改下代码，然后再手动的打包、手动刷新页面，这样太麻烦了，webpack 也是支持自动打包及刷新页面的。

webpack-dev-server 可以自动的打开页面、自动的打包、自动的刷新、所以在项目中一般采取这种方式。

安装：`yarn add webpack-dev-server`

在 package.json 中增加：

```diff
"scripts": {
    "build": "webpack",
+   "dev": "webpack-dev-server"
}
```

webpack.config.js 中添加：

```diff
module.exports = {
+    devServer: {
+        open: true  // 自动打开页面
+    }
}
```

![](/caisr.github.io/database/images/articles/webpack/es6/image.png)

页面就是在本地的服务中打开，注意使用 webpack-dev-server 不会生成打包后的目录，它将文件都放在内存中。

还有一个比较常用的配置 proxy，proxy 可以将你的请求进行代理，避免出现跨域问题。

举个例子：

ajax.js

```
const ajax = () => {
  const xhr = new XMLHttpRequest()
  const url = '/api/wikisecond/lemmasecond?lemmaId=10629668'
  xhr.open('get', url)
  xhr.onreadystatechange = function () {
    if(xhr.readyState === 4 && (xhr.status === 200 || xhr.status === 304)) {
      console.log(JSON.parse(xhr.responseText))
    }
  }
  xhr.send()
}

export default ajax
```

index.js

```diff
+ import ajax from './ajax'
+ ajax()
```

webpack.config.js

```diff
devServer: {
    open: true,
    proxy: {
        '/api': {
            target: 'https://baike.baidu.com',
            changeOrigin: true,
            secure: false,
        }
    }
}
```

上面配置的意思就是将`http://localhost:8080/api/xxxx`代理到`https://baike.baidu.com/api/xxx`

重新打包结果：

![](/caisr.github.io/database/images/articles/webpack/es6/image1.png)

直接拿到了百度百科的接口返回内容，没有出现跨域。

### 二. 模块热替换

模块热替换的意思就是在不重新刷新页面的情况下，替换模块，举个例子：

addItem.js

```
import style from './style/index.scss'

export default function () {
  const button = document.createElement('button')
  button.textContent = 'add Item'
  button.onclick = () => {
    const item = document.createElement('p')
    item.textContent = 'item'
    item.classList.add(style.item)
    document.body.appendChild(item)
  }
  document.body.appendChild(button)
}
```

index.js

```diff
+ import addItem from './addItem'
+ addItem()
```

index.scss

```diff
.item {
  &:nth-of-type(odd) {
    background: pink;
  }
}
```

由于之前配置了 css-loader 的 modules 导致 css 名很难辨认，这里优化一下：

```diff
{
   loader: 'css-loader',
   options: {
       importLoaders: 2,
-       modules: true
+       modules: {
+           localIdentName: '[name]_[local]_[hash:base64]'
+       }
   }
}
```

打包后：

![](/caisr.github.io/database/images/articles/webpack/es6/image2.png)

每点击一下 add Item 按钮就会在页面上增加一个 item 元素，奇数元素的背景色为粉色。

如果这时候改一下背景色：

index.scss

```diff
.item {
   &:nth-of-type(odd) {
-    background: pink;
+    background: orange;
   }
 }
```

页面会重新刷新，那么之前添加的 item 就都没了。

![](/caisr.github.io/database/images/articles/webpack/es6/image3.png)

又得去点击 add Item 按钮才行，如果想保持之前的页面状态，那么就要用到模块热替换功能：

wenpack.config.js

```diff
+ const webpack = require('webpack')

devServer: {
    open: true,
    proxy: {
      '/api': {
        target: 'https://baike.baidu.com',
        changeOrigin: true,
        secure: false,
      }
    },
+   hot: true,
+   hotOnly: true
},
plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve('template', 'index.html')
    }),
    new CleanWebpackPlugin(),
+   new webpack.HotModuleReplacementPlugin()
]
```

hot 就是开启模块热替换，hotOnly 是告诉 webpack 在模块热替换失败的时候不要重新刷新页面。

但是关于 js 代码的热更新就有点麻烦，得这样写：

index.js

```
if (module.hot) {
    module.hot.accept('变动的模块路径', function () {
      // 做一些事情
    })
}
```

### 三. 处理 ES6 语法

虽然 ES6 在很早就发布了，但是还是有浏览器没有实现 ES6 中的部分语法，所以需要对 ES6 的语法进行降级，要用到的就是 babel 了。

找到官方推荐配置，[webpack](https://babeljs.io/setup#installation)

1. 安装相关包

   ```
   yarn add babel-loader @babel/core @babel/preset-env -D
   ```

2. webpack.config.js 中添加

   ```diff
   module: {
       rules: [
   +       { test: /\.js$/, exclude: /node_modules/, loader: 'babel-loader' }
       ]
   }
   ```

3. 创建`.babelrc`文件：

   ```
   {
       "presets": ["@babel/preset-env"]
   }
   ```

4. 测试一下：

   es6-test.js

   ```
   export default async () => {
       await new Promise(resolve => {
           setTimeout(() => {
               console.log('step 1')
               resolve()
           }, 1000)
       })
       console.log('step 2')
   }
   ```

   index.js

   ```
   import es6Test from './es6-test'
   es6Test()
   ```

   打包后：

   ![](/caisr.github.io/database/images/articles/webpack/es6/image4.png)

   为何按照官网的配置也会出错！！！

   搜索了一下，发现了新世界，babel 居然也这么难配置？看了很多教程，我觉得这个写的最好[Babel 快速上手使用指南](https://juejin.im/post/6844903858632654856#heading-5)，那么就按照这篇来配置：

   `yarn add @babel/plugin-transform-runtime -D`

   `yarn add @babel/runtime core-js@3`

   .babelrc

   ```diff
   {
        "presets": [
            ["@babel/preset-env", {
                "modules": false,
                "useBuiltIns": "usage",
                "corejs": 3
            }]
        ],
        "plugins": ["@babel/plugin-transform-runtime"]
    }
   ```

   重新打包，终于可以了。

   ![](/caisr.github.io/database/images/articles/webpack/es6/image5.png)

   开发模式打包的大小：

   ![](/caisr.github.io/database/images/articles/webpack/es6/image6.png)

   正式模式打包的大小：

   ![](/caisr.github.io/database/images/articles/webpack/es6/image7.png)

完整的配置：[GitHub](https://github.com/GreedyWhale/webpack_demo/tree/HMR_AND_ES6)
