最近工作中有用到服务端渲染，一边学一边写了一个小 demo，这里记录下过程：

服务端渲染的好处就不多说了，网上有很多，直接开始吧！

- ### 使用 create-nuxt-app

  [官网](https://zh.nuxtjs.org/guide/installation)

  按照官网的教程第一步：

  `yarn create nuxt-app <项目名>`

  我的项目名也叫 nuxt-app 所以我输入的命令是：

  `yarn create nuxt-app nuxt-app`

- ### 选择项目配置

  输入完上面的命令后会有一些选项：

  ![](/caisr.github.io/database/images/articles/vue/nuxt/image.png)

  需要注意的是 choose rendering mode 这一项，它有 Universal 和 SPA 这两种选项，如果选了 SPA 那么构建完成之后，生成包含常见的 meta 和资源链接，但不包含页面内容。

  这里坑了我好久，别人的项目中，请求到的 html 都有渲染好的内容在里面，我选了这个模式后就和用 vue-cli 构建的单页面应用一样，只有基本的结构。最后还是在文档中找到原因。

  ![](/caisr.github.io/database/images/articles/vue/nuxt/image1.png)

- ### 添加 css 预处理器

  我用 scss，所以手动添加（其他的也一样，都要手动添加）

  `yarn add node-sass sass-loader -D`

- ### 解决跨域问题

  安装官方提供的包

  `yarn add @nuxtjs/proxy`

  然后在 nuxt.config.js 中找到 axios 这一项，加上 proxy: true 属性

  ![](/caisr.github.io/database/images/articles/vue/nuxt/image2.png)

  然后添加

  ![](/caisr.github.io/database/images/articles/vue/nuxt/image3.png)

  '/guiderank-web'就是要匹配的请求

  'https://zone.guiderank-app.com' 这一部分就是要代理去的接口域名

  这个接口是我随便找的，也可以用其他

  在组件中这样用

  ![](/caisr.github.io/database/images/articles/vue/nuxt/image4.png)

  asyncData 这个是 nuxt 提供的生命周期，文档上写的很详细

  结果：

  ![](/caisr.github.io/database/images/articles/vue/nuxt/image5.png)

  成功拿到

  不过这个接口的 access-control-allow-origin 是\*，我又去 A 站上随便找了个接口，测试了下，用这个方法是可以拿到的

  ![](/caisr.github.io/database/images/articles/vue/nuxt/image6.png)

  ![](/caisr.github.io/database/images/articles/vue/nuxt/image7.png)

- ### 开发页面

  正常的应该要封装一下 axios，我这里就不封装了。（懒）

  开发页面就是用 vue 的语法，但是需要注意的有：

  自己写的一些库，需要通过

  ![](/caisr.github.io/database/images/articles/vue/nuxt/image8.png)

  这种方法引入。

  全局的样式文件通过这种方式引入：

  ![](/caisr.github.io/database/images/articles/vue/nuxt/image9.png)

  其他的知识点就需要去看文档了。

  最后还有个坑，我部署到 gitPages 中 js 路径出问题，找不到文件，目前还没找到原因。
