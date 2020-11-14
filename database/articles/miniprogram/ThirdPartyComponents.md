最近在开发微信小程序的组件库，这里就将我在开发过程中遇到的一些问题记录一下。

## 一. 项目初始化

令人高兴的是，小程序官方提供了一个脚手架工具用于一键生成项目，对于我这种畏惧 webpack 的人来说简直就是福音。

- [GitHub 地址](https://github.com/wechat-miniprogram/miniprogram-cli)
- [官方文档](https://developers.weixin.qq.com/miniprogram/dev/framework/custom-component/trdparty.html)
- [在小程序中使用 npm 的文档](https://developers.weixin.qq.com/miniprogram/dev/devtools/npm.html)

#### 1. 安装

打开命令行工具

```

mkdir miniprogram-cli  // 创建存放脚手架工具的目录

cd miniprogram-cli      // 进入该目录

npm install @wechat-miniprogram/miniprogram-cli  // 安装

```

![](/madao.github.io/database/images/articles/mini_program/third_party_components/image.png)

如果查看版本没问题则证明安装成功

#### 2. 使用 miniprogram-cli 初始化一个项目

```
npx miniprogram init -n demo
```

-n 的意思是使用线上最新的模板进行项目的初始化，其他命令参考[GitHub 地址](https://github.com/wechat-miniprogram/miniprogram-cli)

#### 3. 选择项目的配置

当你输入了上面的命令后，会出现一个选项，这个和 vue-cli 挺像的。

- 选择项目类型，选择 custom-component
  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image1.png)
- 输入项目名称，直接回车的话就是`npx miniprogram init -n demo`这个命令中-n 后面的名字，这里就直接回车
  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image2.png)
- 输入包的版本，这里就安装 x.x.x 这种格式输入即可，或者直接回车就是 1.0.0
  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image3.png)
- 输入你组件库打包后的文件名，这里就直接回车，和官方推荐的一致就好了
  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image4.png)
- 输入 GitHub 仓库的地址
  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image5.png)
- 输入作者的信息
  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image6.png)

上面选项选择完毕后就开始生成了

#### 4. 项目目录介绍

![](/madao.github.io/database/images/articles/mini_program/third_party_components/image7.png)

- src: 开发组件的源码，会发布到 npm 上
- test: 测试目录，暂时没用用过测试功能，等用了再来补充
- tools: 项目打包的配置目录
- tools/demo，这个目录是用于开发组件时候，开发人员自己测试组件的目录，等下会详细介绍
- 其余的文件就是一些配置了，如果不熟悉的话，建议阅读相关文档后再进行修改。

## 二. 开始开发

打开 package.json，可以看到 cli 工具已经生成了一些命令。

![](/madao.github.io/database/images/articles/mini_program/third_party_components/image8.png)

但是使用之前，需要先安装依赖。

在刚刚生成的目录下（注意别进错了）输入

```
yarn install
```

这里我选择 yarn 进行安装，因为 npm 偶尔会出现一些奇奇怪怪的问题。

下载完依赖后，打开目录中的 README.md 文件，按照它提供的开发步骤，进行开发

- #### yarn dev

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image34.png)

  这是它关乎 yarn dev 的说明，因为我使用 yarn，所以后面的命令都会用 yarn 进行演示。

  当执行完`yarn dev`后，会发现多了一个目录，用微信开发者工具打开这个目录

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image9.png)

  注意打开的目录

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image10.png)

  打开后就可以看到这样一个页面

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image11.png)

  那行文本就是官方的 demo 了。

  再来看看 miniprogram_dev 目录

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image12.png)

  所以 demo 目录的作用就是用于生成 miniprogram_dev 目录，然后 miniprogram-cli 会把我们写的组件打包后放在`miniprogram_dev/components`目录下。

  这里需要注意的一个问题是：

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image13.png)

  这里引入组件的路径是基于该文件在 miniprogram_dev 目录下的位置，也可以直接写成这样：

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image14.png)

  如果想要边改代码边更新，那么就要用到`yarn watch`命令了，因为 README.md 文件中介绍的比较详细这里就不再赘述了。

  有时你会发现，你改动了文件但是并没有更新，这时候运行`yarn clean-dev`，然后再运行`yarn watch`就好了，建议运行这两条命令前把微信开发者工具关闭，否则很容易出现卡死的情况。

- ### 开启 less

  官方文档上说了，支持 less 编写 wxss，但是默认是关闭的，安装他说的我们去
  `tools/config.js`中开启：

  找到 wxss 的配置，开启 less

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image15.png)

  把官方的 demo 中的 index.wxss 修改一下：

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image16.png)

  第一个坑就来了，你会发现打包报错了：

  ```

  Error: File not found with singular glob: /Users/madao/miniprogram-cli/demo/src/index.wxss (if this was purposeful, use `allowEmpty` option)

  ```

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image17.png)

  反正我是去网上搜，没搜到，文档也没查到，然后我在 tools 目录下找关于 wxss 的配置，然后找到了这样一个配置：

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image18.png)

  把这里的 wxss 改为 less，然后重新`yarn watch`

  报错消失啦，less 文件也成功编译成了 wxss 文件。

  第二个坑，你如果这时候再去改这个 less 文件，你会发现它不会自动更新到 miniprogram_dev 目录

  再去 tools 里面找，找到这个：

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image19.png)

  把这里也改成 less，然后重新`yarn watch`

  到这里 less 相关的就搞定了

- ### 多入口

  因为是组件库，肯定不能只写一个组件，但是我查看官方的例子并没有多组件的例子，只能自己去摸索了，miniprogram-cli 使用的是 gulp + webpack，所以最终还是逃不过 webpack，去搜索有关 webpack 多入口打包相关的教程，找到了这样一个函数：

  ```

  const getEntry = () => {

      const globPath = 'src/components/**/*.js' // 匹配src目录下的所有文件夹中的html文件

      // (\/|\\\\) 这种写法是为了兼容 windows和 mac系统目录路径的不同写法

      /* eslint-disable no-useless-escape */

      const pathDir = 'src(\/|\\\\)(.*?)(\/|\\\\)' // 路径为src目录下的所有文件夹

      const files = glob.sync(globPath)

      const entries = []

      const reg = new RegExp('^' + pathDir)

      for (let i = 0; i < files.length; i++) {

          entries.push(files[i].replace(reg, '$`').replace('.js', ''))

      }

      return entries

  }

  ```

  这是我自己改动过的，[原文在这里](https://juejin.im/post/6844903812885381128)。虽然注释上有说兼容 windows，但是我并没有在 windows 上测试过。

  用上面的函数修改入口配置：

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image20.png)

  由于该函数还依赖了 glob 这个包，所以要去下载它：

  ```
  yarn add glob --dev
  ```

  然后把 src 目录修改成这样

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image21.png)

  去 tools/demo 中改一下组件的引用

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image22.png)

  重新打包看下效果：

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image23.png)

  miniprogram_dev 这里也会生成两个组件

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image24.png)

  到这里开发的相关配置就完了。

## 三. 在项目中使用第三方组件

按照上面的步骤，现在已经有两个组件了，这时候就要发布了，发布之前一定要执行
`yarn dist`这样就会生成 miniprogram_dist 目录，这就是要发布的目录。

发布相关的没有什么好说的，看看 npm 文档或者微信的文档就 ok 了，我这里碰到的就是包名重复问题，也很好解决改个名字就 ok 了，然后把包改成公开，npm 都会有提示的。

现在，我们在其他项目中使用下别的库：

- 在微信开发者工具中创建一个项目，appid 使用测试好就行了

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image25.png)

- 使用官方的提供的例子试一下，[slide-view](https://github.com/wechat-miniprogram/slide-view)
- 在新建的项目中生成 package.json

  `npm init -y`

- 安装 slide-view 库：

  `yarn add miniprogram-slide-view`

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image26.png)

  注意：这里一定要把依赖的第三方库放在 dependencies 字段中，否则会构建失败的。

- 在开发者工具中开启，npm 构建

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image27.png)

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image28.png)

  构建成功后你的目录下会多一个这个目录：

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image29.png)

- 使用第三方库

  引入组件

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image30.png)

  使用

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image31.png)

  以上就是单个组件的组件库使用。

  多个组件的组件库的使用则是下面这样：

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image32.png)

  效果：

  ![](/madao.github.io/database/images/articles/mini_program/third_party_components/image33.png)

以上，就是我经历的开发流程了。
