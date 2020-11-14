在使用 webpack 搭建项目失败了之后，还是选择了 vue-cli 来搭建。下面就把使用 vue-cli 搭建项目的步骤记录一下：

### 一、全局安装 vue-cli

```

npm install -g @vue/cli

# OR

yarn global add @vue/cli

```

（如果没有安装 node.js 还需要安装[node.js](https://nodejs.org/zh-cn/)）

### 二、使用 vue-cli 初始化一个项目

1.  vue create my-project

2.  然后就会有一些选项让你选择：

    - 项目使用默认配置还是手动配置

      ![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image.png)

    一般选择第二项，手动配置

    - 项目依赖安装

      ![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image1.png)

    默认提供的就是上面列出来的这些，按需选择即可。

    [Progressive Web Apps](https://developer.mozilla.org/zh-CN/Apps/Progressive)：这个是渐进式 Web 应用程序

    Unit Testing，E2E Testing：这两个是测试相关的。

    Linter / Formatter：这个是代码的风格检测。

    CSS Pre-processors：css 预处理器，scss，stylus，less 之类的

- 预处理类型选择

  ![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image2.png)

- 选择一个代码风格检测规范

  ![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image3.png)

- 在什么时候进行代码风格检测

  ![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image4.png)

- 将 bable，postcss，eslint 等配置写在 package.json 还是单独文件

  ![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image5.png)

- 是否保存为默认配置

  ![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image6.png)

如果是第一次好像还会问你使用 npm 还是 yarn 作为依赖的下载工具，我这是第二次了，所以没有提示。

选择完看到下图，就证明初始化成功了

![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image7.png)

上面这些步骤中可能失败的步骤是下载 node-sass 失败，要么科学上网，要么用国内的源，实在不行就用 yarn 进行下载。

启动项目的时候我还碰到这个问题

![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image8.png)

好像是 npm 包安装不完整，重新装一遍 npm 包就好了`npm i`

### 三、安装 ElementUI 和 axios

ElementUI 是一个样式的框架，axios 则是用来发送请求的一个库。

```
npm i axios element-ui
```

装好了这两个，还需要装 babel-plugin-transform-runtime，装这个是因为我在封装请求的时候，在 js 文件中不能写箭头函数等 es6 的语法，我也不知道装的对不对，装了就可以用了：

```

npm install --save-dev babel-plugin-transform-runtime

npm install --save babel-runtime

```

然后按照 element ui 官网说的方式引入组件，在项目中的 main.js 中写上

```

import 'element-ui/lib/theme-chalk/index.css'

import ElementUI from 'element-ui'



Vue.use(ElementUI)

```

这时候打包看下：

![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image9.png)

可以看到打包后的体积不小，而且还什么都没写就这么大，所以不能全部把全部的 elementUI 组件引入，就要用到按需加载，按照 elementUI 的文档先安装：

```
npm install babel-plugin-component -D
```

然后找到你的 babel 配置，加入下面这些配置：

```

'plugins': [

    [

      'component',

      {

        'libraryName': 'element-ui',

        'styleLibraryName': 'theme-chalk'

      }

    ],

    '@babel/plugin-transform-runtime'

  ]

```

![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image10.png)

然后在 main.js 中把

```

import ElementUI from 'element-ui'

import 'element-ui/lib/theme-chalk/index.css'



Vue.use(ElementUI)

```

删掉，因为按照官网的文档，要这样写

```

import { Button, Select } from 'element-ui';



Vue.component(Button.name, Button);

Vue.component(Select.name, Select);

```

有一个问题就是当你的组件变得多的时候，你的 main.js 就会变得很长，所以我将用到的组件单独写在一个 js 文件中。如图：

![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image11.png)

在 main.js 中写上下面图中的代码

![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image12.png)

这还有个问题，就是类似于这种，所以目前这个组件的配置列表 js 文件中只能放组件不能放方法：

![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image12.png)

我目前还没用到这些，所以先不说，等用到了我再更新上去。

然后在首页加一个组件试下

![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image13.png)

启动项目看下：

![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image14.png)

ok，这里组件正常的的添加了，然后看下打包体积：

![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image15.png)

小了很多，但是其实还是有点大（这是开发环境打包，正式的还会小一点），由于对 webpack 用的很不熟练，所以暂时先这样，等我再研究研究。

### 四、请求封装

直接上代码吧，我也不知道要怎样说，我觉得封装的不好，超时部分借鉴（抄袭）了这里[Adding Retry Paramete](https://github.com/axios/axios/issues/164#issuecomment-327837467)：

```

// index.js， 这里的请求数据类型使用了formData类型

import axios from 'axios'

import qs from 'qs'

import { filterResponse } from './filterResponse'

import { filterStatusCode } from './filterStatusCode'

import { closeLoading } from './closeLoading'

import { Loading } from 'element-ui'

import { TOKEN } from '../constant'



let isLoading



const Axios = axios.create({

  baseURL: location.origin,

  timeout: 10000,

  responseType: 'json',

  withCredentials: true,

  headers: {

    'Content-Type': 'application/x-www-form-urlencoded;charset=utf-8'

  },

  transformRequest: [data => qs.stringify(data)]

})



Axios.defaults.retry = 3

Axios.defaults.retryDelay = 10000



Axios.interceptors.request.use(

  config => {

    isLoading = config.isLoading || false

    if (!isLoading) {

      isLoading = Loading.service({

        text: '拼命加载中',

        background: 'rgba(0, 0, 0, 0.8)'

      })

    }

    if (window.localStorage.getItem(TOKEN)) {

      config.headers.Authorization = window.localStorage.getItem(TOKEN)

    }

    return config

  },

  error => {

    return Promise.reject(error)

  }

)

Axios.interceptors.response.use(

  response => {

    isLoading = closeLoading(isLoading)

    return filterResponse(response)

  },

  error => {

    if (error.message && error.message.match(/timeout/)) {

      const config = error.config

      if (!config || !config.retry || config.method === 'post') {

        isLoading = closeLoading(isLoading)

        return Promise.reject(error)

      }

      config.__retryCount = config.__retryCount || 0

      if (config.__retryCount >= config.retry) {

        isLoading = closeLoading(isLoading)

        return Promise.reject(error)

      }

      config.__retryCount += 1

      const backOff = new Promise(function (resolve) {

        setTimeout(function () {

          resolve()

        }, config.retryDelay || 1)

      })

      return backOff.then(function () {

        return Axios(config)

      })

    }

    isLoading = closeLoading(isLoading)

    filterStatusCode(error)

    return Promise.reject(error)

  }

)



export default {

  install: function (Vue, Option) {

    Object.defineProperty(Vue.prototype, 'ajax', { value: Axios })

  }

}



```

```

// filterResponse.js，stat是和后端定义的接口状态，可自行替换

import Router from 'vue-router'

import { showDialog } from '../dialog'



const router = new Router()



const filterResponse = response => {

  if (response.data.stat) {

    switch (response.data.stat) {

      case 1:

        return Promise.resolve(response.data)

      case 2:

        showDialog(response.data.msg)

        return Promise.reject(response.data)

      case 13:

        return router.push({ path: '/login' })

      default:

        showDialog(response.data.msg)

        return Promise.reject(response.data)

    }

  }

  return Promise.resolve(response.data)

}



export {

  filterResponse

}



```

```

// filterStatusCode.js

import { showDialog } from '../dialog'



const filterStatusCode = error => {

  if (error && error.response) {

    switch (error.response.status) {

      case 404:

        return showDialog('请求错误,未找到该资源')

      case 500:

        return showDialog('服务器端出错')

      case 503:

        return showDialog('服务不可用')

      default:

        return showDialog('恭喜你，获得解锁神秘错误成就')

    }

  } else {

    return showDialog('请求失败')

  }

}



export {

  filterStatusCode

}



```

```

// closeLoading.js

const closeLoading = (isLoading) => {

  if (isLoading) {

    isLoading.close()

    isLoading = null

  }

  return isLoading

}

export { closeLoading }



```

```

// main.js

import AxiosPlugin from './utils/request/index'

Vue.use(AxiosPlugin)

```

### 五、环境切换

在正常的开发中我们肯定是分几种环境的，比如正式，测试之类的，在 vue-cli 中我们可以新建这几个文件

![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image16.png)

每一个文件的代表开发，生产，预发布，当然这些名字都是可以自定义的，这些文件有什么用呢，比如我们不同环境开发使用的域名不同，我们就可以放在这些文件中，比如在.env.development 文件中：
![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image17.png)

然后在封装请求的 js 文件中这样用

![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image18.png)

这里注意变量的命名要加上 VUE_APP，[文档链接](https://cli.vuejs.org/guide/mode-and-env.html#example-staging-mode)。

然后在 package.json 中打包命令也要修改成下面这样：

```

"scripts": {

    "dev": "vue-cli-service serve --mode development",

    "stag": "vue-cli-service serve --mode staging",

    "prod": "vue-cli-service serve --mode production",

    "dev-build": "vue-cli-service build --mode development",

    "stag-build": "vue-cli-service build --mode staging",

    "prod-build": "vue-cli-service build --mode production",

    "lint": "vue-cli-service lint"

  },

```

--mode 用来切换环境，后面跟的字符串就是之前写的.env 的文件名

### 六、修改 webpack 配置

如果要修改默认的 webpack 配置，需要在根目录下新建一个 vue.config.js，然后在里面写配置，[文档链接](https://cli.vuejs.org/guide/webpack.html#simple-configuration)

查看默认配置：

![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image19.png)

就是这几个文件了。

可配置项，可以在这里看，也可以去文档里找
![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image21.png)

### 七、路由懒加载

直接看文档，中文的(〃'▽'〃)[文档地址](https://router.vuejs.org/zh/guide/advanced/lazy-loading.html#%E6%8A%8A%E7%BB%84%E4%BB%B6%E6%8C%89%E7%BB%84%E5%88%86%E5%9D%97)

动态的改变页面 title

我是用的这种方式

```

/*

专门存放路由配置的文件，可自己新建，由于我这个项目准备做后台管理所以

我的主要路由就一个，其他都是嵌套的路由

*/

const Layout = () => import(/* webpackChunkName: "layout" */ '../views/layout/layout.vue')

const StatisticsScm = () => import(/* webpackChunkName: "statisticsScm" */ '../views/statistics/scm/scm.vue')

const domain = process.env.VUE_APP_DOMAIN



export default [

  {

    path: `/${domain}`,

    name: 'Layout',

    component: Layout,

    children: [

      {

        path: `/${domain}/statistics/scm/index`,

        name: 'StatisticsScm',

        component: StatisticsScm,

        meta: {

          title: '数据统计'

        }

      }

    ]

  }

]



```

```

// router.js

import Vue from 'vue'

import Router from 'vue-router'

import routes from './config/routeConfig'



Vue.use(Router)

const router = new Router({

  mode: 'history',

  routes

})



router.beforeEach((to, from, next) => {

  window.document.title = to.meta.title || ''   //这里就是用来设置页面的title的

  next()

})



export default router

```

### 八、代理

如何绕过同源策略的限制

我是差不多自己摸索出来的，估计有问题，我自己用着还行。。。直接贴代码

在 vue.config.js 中，如果没有要新建一个在根目录

```



module.exports = {

 baseUrl: `/${process.env.VUE_APP_DOMAIN}/`,

 devServer: {

   open: true,

   port: 3000,

   hot: true,

   https: true,

   host: 'localhost',

   compress: true,

   proxy: {

     '/api': {

       target: 'https:xxxxx',   // 需要代理去的接口地址

       changeOrigin: true,     // 跨域请求必须开启

       secure: false,                 // 不要检查证书

       pathRewrite: {

         '^/api': ''

       },

       bypass: function (req, res, proxyOptions) {            // 这里就是只代理接口，不代理页面，我在做的时候直接把页面也代理去了真实的页面。。

         if (req.headers.accept.indexOf('html') !== -1) {

           console.log('Skipping proxy for browser request.')

           return './public/index.html'

         }

       }

     }

   }

 }

}

```

```

// 请求例子

 Axios.post('/api/login', {

      userName: 'user',

      password: 'password'

 })

```

有时候会出现这种错误：

![image.png](/caisr.github.io/database/images/articles/vue/vue_cli/image20.png)

就是不断的请求这个然后失败，我的解决办法是改下端口号，或者 host 值改为 localhost，然后重启项目。如果谁有更好的解决方法，麻烦告诉我一下，感激不尽

写的有点乱，这些就是我用 vue-cli 搭建项目的所有步骤了。。还是得学 webpack 啊，唉。。。
