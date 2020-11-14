### 一. Tree Shaking

tree shaking 就是不要打包没有用到的代码，接着上一篇的配置，举个例子：

methods.js

```
export const add = (a, b) => {
  console.log(a + b)
}

export const square = (a) => {
  console.log(a * a)
}
```

index.js

```diff
+ import { add } from './methods'


+ add(1, 2)
```

打包结果：

![](/madao.github.io/database/images/articles/webpack/tree_shaking_mode/image.png)

将 add 和 square 都打包进来了，但是实际只用到了 add，那么 square 就是没有用到的代码了。

要做到只打包 add 不打包 square，根据模式的不同，有不同的配置

- ### development

  当 mode 为 development 的时候，需要在 webpack.config.js 中添加以下配置：

  ```diff
  module.exports = {
  +   mode: 'development',
  +   optimization: {
  +     usedExports: true
  +   }
  }
  ```

  配置好之后重新打包：

  ![](/madao.github.io/database/images/articles/webpack/tree_shaking_mode/image1.png)

  还是存在的，但是会多一个信息：

  ![](/madao.github.io/database/images/articles/webpack/tree_shaking_mode/image2.png)

  所以在 development 模式下，即使配置了也不会把没有用到的代码剔除，所以真正实现的这个功能需要在 production 模式下。

- ### production

  webpack.config.js

  ```diff
   module.exports = {
  +   mode: 'production',
  -   optimization: {
  -     usedExports: true
  -   }
  }
  ```

  是的，在 production 模式下，webpack 会自动的启用 tree shaking，不需要 optimization 配置，改一下模式就好。

  现在看看打包结果

  ![](/madao.github.io/database/images/articles/webpack/tree_shaking_mode/image3.png)

  将所有的 console.log 找到，并没有类似`console.log(a * a)`的代码，那么证明 square 没有被打包进来。

- ### sideEffects

  如果看官方文档呢，他还说了一个 sideEffects 配置，说实话并没有看明白他说的是什么意思，我是这样理解的。

  首先创建一个 helloWorld.js

  ```
  console.log('hello world!')
  ```

  将 index.js 中已有的代码全部注释，方便查看打包后的内容，注释之后，添加以下代码：

  ```diff
  + import './helloWorld'
  ```

  重新打包：

  ![](/madao.github.io/database/images/articles/webpack/tree_shaking_mode/image4.png)

  console.log 就剩这一个了，是符合我的预期的。

  然后按照官网的说明，给 package.json 加一个配置：

  ```diff
  {
  +  "sideEffects": false
  }
  ```

  重新打包：

  ![](/madao.github.io/database/images/articles/webpack/tree_shaking_mode/image5.png)

  hello world 没了，同样的如果以这种方式引用 css 资源：

  `import './style/xxx.scss'`

  那么在最终的打包结果中是不包含 xxx.scss 中的内容的。

  所以如果你的模块是例子中这种写法，没有 export 任何东西，那么在使用 sideEffects 这个配置的时候就要将 sideEffects 的值变成一个数组，告诉 webpack 就算这些模块没有 export 任何东西，也不要将它删除：

  ```diff
  {
  +   "sideEffects": [
  +       "*.scss",
  +       "./src/helloWorld.js"
  +   ]
  }
  ```

### 二. mode

在真实的项目中，需要区分环境进行打包，比如开发环境和正式环境，webpack 中也提供了不同的 mode 用于不同的环境。

新建 webpack.prod.js

```
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
  mode: 'production',
  entry: {
    main: path.resolve('src', 'index.js')
  },
  output: {
    filename: '[name].js',
    path: path.resolve('dist')
  },
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif)$/,
        use: [{
          loader: 'url-loader',
          options: {
            name: '[name]_[hash].[ext]',
            outputPath: path.join('assets', 'images'),
            limit: 1024
          }
        }]
      },
      {
        test: /\.(css|scss)$/,
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              importLoaders: 2,
              modules: {
                localIdentName: '[name]_[local]_[hash:base64]'
              }
            }
          },
          { loader: 'postcss-loader' },
          {
            loader: 'sass-loader',
            options: {
              implementation: require("sass")
            }
          }
        ]
      },
      { test: /\.js$/, exclude: /node_modules/, loader: 'babel-loader' }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve('template', 'index.html')
    }),
    new CleanWebpackPlugin(),
  ]
}
```

创建 webpack.dev.js

```
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const webpack = require('webpack')

module.exports = {
  mode: 'development',
  entry: {
    main: path.resolve('src', 'index.js')
  },
  output: {
    filename: '[name].js',
    path: path.resolve('dist')
  },
  devServer: {
    open: true,
    proxy: {
      '/api': {
        target: 'https://baike.baidu.com',
        changeOrigin: true,
        secure: false,
      }
    },
    hot: true
  },
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif)$/,
        use: [{
          loader: 'url-loader',
          options: {
            name: '[name]_[hash].[ext]',
            outputPath: path.join('assets', 'images'),
            limit: 1024
          }
        }]
      },
      {
        test: /\.(css|scss)$/,
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              importLoaders: 2,
              modules: {
                localIdentName: '[name]_[local]_[hash:base64]'
              }
            }
          },
          { loader: 'postcss-loader' },
          {
            loader: 'sass-loader',
            options: {
              implementation: require("sass")
            }
          }
        ]
      },
      { test: /\.js$/, exclude: /node_modules/, loader: 'babel-loader' }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve('template', 'index.html')
    }),
    new CleanWebpackPlugin(),
    new webpack.HotModuleReplacementPlugin()
  ]
}
```

这样就有两个配置文件了，然后修改 package.json 中的打包命令

```diff
"scripts": {
-    "build": "webpack",
-    "dev": "webpack-dev-server"
+    "build": "webpack --config webpack.prod.js",
+    "dev": "webpack-dev-server --config webpack.dev.js"
},
```

但是上面的配置 90%都是相同的，可以将这些重复的配置抽取出来，便于维护：

新建 webpack.common.js

```
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
  entry: {
    main: path.resolve('src', 'index.js')
  },
  output: {
    filename: '[name].js',
    path: path.resolve('dist')
  },
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif)$/,
        use: [{
          loader: 'url-loader',
          options: {
            name: '[name]_[hash].[ext]',
            outputPath: path.join('assets', 'images'),
            limit: 1024
          }
        }]
      },
      {
        test: /\.(css|scss)$/,
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              importLoaders: 2,
              modules: {
                localIdentName: '[name]_[local]_[hash:base64]'
              }
            }
          },
          { loader: 'postcss-loader' },
          {
            loader: 'sass-loader',
            options: {
              implementation: require("sass")
            }
          }
        ]
      },
      { test: /\.js$/, exclude: /node_modules/, loader: 'babel-loader' }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve('template', 'index.html')
    }),
    new CleanWebpackPlugin()
  ]
}
```

然后将 webpack.prod.js 和 webpack.dev.js 中重复的部分删除：

webpack.prod.js

```
module.exports = {
  mode: 'production'
}
```

webpack.dev.js

```
const webpack = require('webpack')

module.exports = {
  mode: 'development',
  devServer: {
    open: true,
    proxy: {
      '/api': {
        target: 'https://baike.baidu.com',
        changeOrigin: true,
        secure: false,
      }
    },
    hot: true
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ]
}
```

然后需要安装一个包，`webpack-merge`，用它将 webpack.common.js 和对应环境的配置合并在一起：

`yarn add webpack-merge -D`

webpack.prod.js

```
const merge = require('webpack-merge')
const common = require('./webpack.common.js')

const prodConfig = {
  mode: 'production'
}
module.exports = merge(common, prodConfig)
```

webpack.dev.js

```
const webpack = require('webpack')
const merge = require('webpack-merge')
const common = require('./webpack.common.js')

const devConfig = {
  mode: 'development',
  devServer: {
    open: true,
    proxy: {
      '/api': {
        target: 'https://baike.baidu.com',
        changeOrigin: true,
        secure: false,
      }
    },
    hot: true
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ]
}
module.exports = merge(common, devConfig)
```

这样一来就区分了开发环境和正式环境，虽然目前看起来没什么必要：），例子中的项目太简单了。

完整代码[GitHub](https://github.com/GreedyWhale/webpack_demo/tree/treeShaking_and_mode)
