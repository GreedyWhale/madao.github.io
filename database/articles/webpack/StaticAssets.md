虽然之前用过 webpack，但基本都是边用边查，因为 webpack 的配置项实在是太多了，还是自己总结下，以后用到了也有个参考。

### 一. 安装

```
yarn add webpack webpack-cli -D
```

### 二. 项目目录

![](/caisr.github.io/database/images/articles/webpack/static_assets/image.png)

- a.js
  ```
  export default () => console.log('~~~~~~~~~~a~~~~~~~~~~~')
  ```
- b.js
  ```
  export default () => console.log('~~~~~~~~~~b~~~~~~~~~~~')
  ```
- c.js
  ```
  export default () => console.log('~~~~~~~~~~c~~~~~~~~~~~')
  ```
- index.js

  ```
  import a from './a'
  import b from './b'
  import c from './c'

  a()
  b()
  c()
  ```

### 三. 创建 webpack.config.js

先从最基础的来只定义入口和出口，webpack 不愧是业界最流行的打包工具，光出口的配置就有十几项：

```
const path = require('path')

module.exports = {
  mode: 'development',
  entry: {
    main: path.resolve('src', 'index.js')
  },
  output: {
    filename: '[name].js',
    path: path.resolve('dist')
  }
}
```

- mode：模式，一共有三种：development，production，none

  虽然 webpack 官网列出了一个表格说明这三种模式的区别，但是得看表格中不同模式开启的插件是干什么的才能真正的明白不同模式的区别。我并没有去看，所以就不多了免得误导，不过看不同模式打包出的代码，可以确定的一点就是：development 模式没有对代码进行压缩，production 模式对代码进行了压缩。

- entry：入口配置，可以有多个入口文件。

- output：出口配置

  - filename：打包后文件的名字，[name]是占位符表示 entry 这个对象中的 key 值，在这个例子中就是 main。

  - path：打包后文件存放的目录

创建好 webpack.config.js 后，给 package.json 中添加打包命令：

```
"scripts": {
    "build": "webpack"
},
```

执行`yarn build`，不出意外的话就可以在项目中看到多了一个 dist 目录了

### 四. 图片资源打包

图片资源的打包要用到 loader，loader 是 webpack 用来对模块的源码进行转换的，这也让 webpack 具有了打包除了 JavaScript 文件之外的文件的能力。

图片资源打包可以使用`file-loader`或者`url-loader`

`url-loader`可以设置 limit，图片如果小于 limit 的值，那么图片会被打包成 base64 格式。

这两个 loader 都是需要手动安装的：

`yarn add url-loader file-loader -D`

- file-loader 配置

  ```
  module: {
      rules: [
          {
              test: /\.(png|jpe?g|gif)$/,
              use: [{
                  loader: 'file-loader',
                  options: {
                      name: '[name].[ext]',
                      outputPath: path.join('assets', 'images')
                  }
              }]
          }
      ]
  }
  ```

  - test: 文件匹配的正则
  - use：表示处理这个文件所需要的 loader，可以有多个，当有多个的时候，loader 的执行顺序是从后往前的，比如：`[a-loader, b-loader]`，是先执行 b-loader，再执行 a-loader。
  - options 就是 loader 的配置了
    - name: 打包后文件的名字，[name].[ext]的意思就是打包后文件名保持原来的文件，[ext]是文件后缀的占位符
    - outputPath：打包后文件位置。

  加上一个图片试一试：

  index.js

  ```
  import avatar from './assets/images/avatar.jpeg'

  const img = new Image()
  img.src = avatar
  document.body.appendChild(img)
  ```

  index.html

  ```
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <meta http-equiv="X-UA-Compatible" content="ie=edge">
      <title>Document</title>
  </head>
  <body>
      <p>测试</p>
      <script src="./dist/main.js"></script>
  </body>
  </html>
  ```

  打包后的结果：

  ![](/caisr.github.io/database/images/articles/webpack/static_assets/image1.png)

  图片路径有问题，因为 index.html 并不位于 dist 目录下，webpack 提供了生成 html 文件的插件，这里暂时先不用，改一下 loader 的配置：

  ```diff
  options: {
      name: '[name].[ext]',
      outputPath: path.join('assets', 'images'),
  +   publicPath: path.join('..', 'dist', 'assets', 'images')
  }
  ```

  publicPath 的值可以认为是 img 标签中的 src 中除了图片名以外的路径，现在就可以看到图片了：

  ![](/caisr.github.io/database/images/articles/webpack/static_assets/image2.png)

- url-loader 配置:

  ```diff
  rules: [
    {
      test: /\.(png|jpe?g|gif)$/,
      use: [{
  -     loader: 'file-loader',
  +     loader: 'url-loader',
        options: {
          name: '[name].[ext]',
          outputPath: path.join('assets', 'images'),
          publicPath: path.join('..', 'dist', 'assets', 'images'),
  +       limit: 10240
        }
      }]
    }
  ]
  ```

  - limit：小于 limit 值的图片都会转成 base64

  看下效果

  ![](/caisr.github.io/database/images/articles/webpack/static_assets/image3.png)

### 五. css 资源打包

css 资源打包需要用到 style-loader、css-loader，首先安装它们：

```
yarn add style-loader css-loader -D
```

```
{
    test: /\.css$/,
    use: [{ loader: 'style-loader' }, { loader: 'css-loader' }]
}
```

注意顺序，css-loader 是将引用到的 css 文件合并成一段 css，style-loader 是将合并好的 css 放在 style 标签内。

index.css：

```
p {
  color: blueviolet;
}
```

index.js

```
import './style/index.css'
```

打包后的结果：

![](/caisr.github.io/database/images/articles/webpack/static_assets/image4.png)

现在大家基本都用 css 预处理器来写 css 了，所以还得处理这些，用 scss 来举例：

处理 scss 格式的文件需要用到`sass-loader node-sass`，但是 node-sass 这个包经常性会安装失败，所以这里选用 dart-sass，

`yarn add sass-loader sass -D`

```diff
{
    test: /\.(css|scss)$/,
    use: [
        { loader: 'style-loader' },
        { loader: 'css-loader' },
+       {
+           loader: 'sass-loader',
+           options: {
+             implementation: require("sass")
+           }
+       }
    ]
}
```

将 index.css 改为 index.scss

```
body {
  p {
    color: blueviolet;
  }
}
```

将 index.js 中的引入也改一下：

```
import './style/index.scss'
```

打包结果：

![](/caisr.github.io/database/images/articles/webpack/static_assets/image5.png)

css 资源的打包还有一个要注意的，就是浏览器厂商前缀:

![](/caisr.github.io/database/images/articles/webpack/static_assets/image6.png)

用 postcss-loader 这个 loader 就可以自动的为 css 属性加上前缀了，在这之前还需要下载一个插件`autoprefixer`。

`yarn add postcss-loader autoprefixer -D`

接着创建一个 postcss 配置文件 postcss.config.js:

```
module.exports = {
  plugins: [
    require('autoprefixer')
  ]
}
```

还要一个浏览器兼容列表，`.browserslistrc`:

```
> 1%
last 2 versions
not ie <= 8
```

webpack.config.js

```diff
{
    test: /\.(css|scss)$/,
    use: [
        { loader: 'style-loader' },
        { loader: 'css-loader' },
+       { loader: 'postcss-loader' },
        {
            loader: 'sass-loader',
            options: {
              implementation: require("sass")
            }
        }
    ]
}
```

然后在 index.scss 中加上一些动画：

```
@keyframes rotate {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

img {
  animation: rotate 1s ease-in infinite;
}
```

打包看结果：

![](/caisr.github.io/database/images/articles/webpack/static_assets/image7.png)

到了这里，渐渐的将就开始坑了，你会发现越配越多，不仅要知道 webpack 的配置，还得知道相关 loader 的配置，绝望。。

继续配置。

接下来配置 css 的@import 这种情况的处理：

将 index.scss 中动画相关的 css 提取到 animate.scss，并在 index.scss 中导入它：

```
@import './animation.scss';
```

直接打包：

可以看到它并没有经过 postcss-loader 的处理：

![](/caisr.github.io/database/images/articles/webpack/static_assets/image8.png)

为了让 webpack 可以正确处理它，需要加上下面配置：

```
{
    loader: 'css-loader',
    options: {
        importLoaders: 2
    }
}
```

> importLoaders：用于配置「css-loader 作用于 @import 的资源之前」有多少个 loader。

上面是中文网的解释，挺直白的就直接抄了。

css-loader 还有一个常用到的配置 modules：

```
{
    loader: 'css-loader',
    options: {
        importLoaders: 2,
        modules: true
    }
}

```

给 a.js 添加以下代码：

```
import style from './style/index.scss'

const name = document.createElement('p')
name.textContent = 'allen'
name.classList.add(style.name)
document.body.appendChild(name)
```

index.js:

```diff
- import './style/index.scss'
+ import style from './style/index.scss'

+ img.classList.add(style.avatar)
```

index.scss:

```diff
+ .avatar {
+   border-radius: 50%;
+   width: 100px;
+ }
+ .name {
+   font-size: 40px;
+   color: orange;
+  }
```

打包看结果：

![](/caisr.github.io/database/images/articles/webpack/static_assets/image9.png)
modules 就是让 class 不冲突的一个配置。

### 六. 自动生成 html

自动生成 html 文件，要用到 webpack 的一个插件`html-webpack-plugin`，需要手动安装：`yarn add html-webpack-plugin -D`

在 webpack.config.js 中添加：

```diff
+ const HtmlWebpackPlugin = require('html-webpack-plugin')
module.exports = {
// 之前配置的省略
+   plugins: [new HtmlWebpackPlugin()]
}
```

这时候打包就可以看到，dist 目录下多了 index.html，并且自动的引用了打包后的 js 文件

![](/caisr.github.io/database/images/articles/webpack/static_assets/image10.png)

html-webpack-plugin 所做的事就是：自动生成一个 html 文件，并把打包后的 js 文件自动引入到这个 html 文件中。

也可以指定一个模板，把 template 目录下的 index.js 中的 script 标签删除，然后把这个 html 当做模板。

webpack.config.js

```diff
plugins: [
    new HtmlWebpackPlugin({
+     template: path.resolve('template', 'index.html')
    })
]
```

生成的 html 文件就会有模板中的所有内容，并且会自动的引入打包后 js 文件。

记得将 url-loader 的配置修改一下：

```diff
use: [{
    loader: 'url-loader',
    options: {
        name: '[name]_[hash].[ext]',
        outputPath: path.join('assets', 'images'),
-       publicPath: path.join('..', 'dist', 'assets', 'images'),
        limit: 1024
    }
}]
```

### 七. 打包之前清理 dist 目录

要清理目录需要用到`clean-webpack-plugin`这个插件，首先安装：

`yarn add clean-webpack-plugin -D`

配置如下：

webpack.config.js

```diff
+ const { CleanWebpackPlugin } = require('clean-webpack-plugin')
  plugins: [
      new HtmlWebpackPlugin({
        template: path.resolve('template', 'index.html')
      }),
+     new CleanWebpackPlugin()
  ]
```

以上就是一些基本的配置，说实话挺麻烦的，无奈 parcel 还有些不成熟。

全部配置可以看这里：[github](https://github.com/GreedyWhale/webpack_demo/blob/static/webpack.config.js)
