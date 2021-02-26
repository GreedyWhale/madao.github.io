春节马上来了，之前的立的 Flag 果然没有完成 🤔，这应该是春节前的最后一篇笔记了，最近没什么动力啊，虽然年后有换工作的想法，可是完全没动力...

好了，开始学习 Next.js 吧

不论是 Express 还是 Koa，在视图层的实现都需要开发者自己进行集成，比如说在开发页面的时候你想用一些框架，都要自己去集成，Next.js 则不同，Next.js 集成了 React、TypeScript、css in js 等等一些前端常用的技术，还支持页面的静态化、预渲染和服务端渲染，但是 Next.js 没有提供数据库及测试方面的功能。

这次从实现一个简单的 Blog 来学习 Next.js，学习完了，正好把我现在这一版的博客用 Next.js 重构了，在使用 Vue3 的过程中逐渐发现 React 更加优雅，我也不知道咋回事 😶

### 一. 項目初始化

```
mkdir next-blog-demo

cd next-blog-demo

yarn create next-app
```

然后会得到如下的目录结构：

![](/madao.github.io/database/images/articles/node/next/image.png)

### 二. 使用 Typescript 进行开发

在当前项目目录下：

```
yarn add typescript @types/react @types/node -D

yarn tsc --init
```

这样会多出一个 tsconfig.json，暂时可以不用改里面的配置，接下来将所有 js 后缀的文件改为 tsx

改完之后会出现很多报错红线，先重新执行一下`yarn dev`，执行完了之后，Next.js 会帮你修改一些 tsconfig.json 中的配置，然后报错红线会消除很多，下面一个一个来解决：

1. \_app.tsx

   在\_app.js 改为\_app.tsx 后会报：

   - `Binding element 'Component' implicitly has an 'any' type.`
   - `Binding element 'pageProps' implicitly has an 'any' type.`

   这时候只要改成这样：

   ```
   import '../styles/globals.css'
   import type { AppProps } from 'next/app'

   function MyApp({ Component, pageProps }: AppProps) {
     return <Component {...pageProps} />
   }

   export default MyApp
   ```

   将 Component 和 pageProps 的类型导入。

   这个\_app.tsx 是干嘛的？文档上有说明[Custom App](https://nextjs.org/docs/advanced-features/custom-app)

   简单说就是入口页面，你在访问对应的页面的时候，页面的组件就会替换代码中的`Component`，所以在这里添加一些样式或者布局会影响所有的页面，比如改成这样：

   ```
   import '../styles/globals.css'
   import type { AppProps } from 'next/app'
   import React from 'react'

   function MyApp({ Component, pageProps }: AppProps) {
     return (
       <React.Fragment>
         <h1>海的那边是敌人</h1>
         <Component {...pageProps} />
       </React.Fragment>
     )
   }
   export default MyApp
   ```

   那么其他页面渲染出来就会在页面顶部多一个`海的那边是敌人`的标题

2. hello.tsx

   这个文件是放在 api 目录下的，也就是常说的获取数据的接口，它报的错是 req 和 res 的类型为 any，只要改成这样：

   ```
   import type {NextApiRequest, NextApiResponse} from 'next'

   export default (req: NextApiRequest, res: NextApiResponse) => {
     res.status(200).json({ name: 'John Doe' })
   }
   ```

这些东西在文档里都有说[nextjs-types](https://nextjs.org/learn/excel/typescript/nextjs-types)

### 三. 支持 SCSS

Next.js 中支持 scss 很简单，只需要安装：

```
yarn add scss -D
```

然后将所有的`.css`后缀改为`.scss`即可。

以`.module.scss`结尾的 scss 文件，Next.js 会自动使用 CSS Modules 进行处理

### 四. 快速导航

Next.js 中有一个内置的组件`Link`，看下如何使用：

1. 在 pages 目录下新建`List.tsx`和`Detail.tsx`

   - List.tsx

     ```
     import { NextPage } from "next"

     const List: NextPage = () => {
       return (
         <div>这是List页面</div>
       )
     }

     export default List
     ```

   - Detail.tsx

     ```
     import { NextPage } from "next"

     const Detail: NextPage = () => {
       return (
         <div>这是Detail页面</div>
       )
     }

     export default Detail
     ```

2. 修改`index.tsx`：

   ```
   import Head from 'next/head'
   import styles from '../styles/Home.module.scss'
   import Link from 'next/link'

   export default function Home() {
     return (
       <div className={styles.container}>
         <Head>
           <title>Create Next App</title>
           <link rel="icon" href="/favicon.ico" />
         </Head>

         <Link href="/List">
           <a>List</a>
         </Link>
         <Link href="/Detail">
           <a>Detail</a>
         </Link>
       </div>
     )
   }
   ```

观察控制台就会发现无论是点击 List 还是 Detail，页面都不会刷新，和常用的`a`标签不一样，点击 List 会请求一个 List.js，点击`Detail`会请求一个`Detail.js`

- 传统导航 -> page1(html,js,css) -> page2(html,js,css)
- 快速导航 -> page1(html,js,css) -> page2(js)

快速导航的优势在于

1. 不会重复的请求相同的资源
2. 页面不会刷新，新页面通过请求新页面的 js 文件
3. 由于节省了一些解析和请求的过程，所以页面切换的速度会很快

### 五. 同构代码

所谓的同构代码就是一份代码可以运行在服务端和客户端，需要注意两点：

1. 不是所有的代码都会执行，比如一个点击事件的回调函数
2. 不是所有的 API 都能用，比如服务端就不能用浏览器提供的 API，反之也一样。

例子：

```
import { NextPage } from "next"

console.log('hello')
const Detail: NextPage = () => {
  return (
    <div>这是Detail页面</div>
  )
}

export default Detail
```

当你访问 detail 页面的时候，在浏览器的控制台以及 Node 的控制台都会输出`hello`

### 六. 全局配置

- #### 自定义 head

  Next.js 提供了 Head 组件，它可以让用户自定义 head

  - 全局 head

    \_app.js

    ```
    import '../styles/globals.scss'
    import type { AppProps } from 'next/app'
    import React from 'react'
    import Head from 'next/head'

    function MyApp({ Component, pageProps }: AppProps) {
      return (
        <React.Fragment>
          <Head>
            <meta name="viewport" content="initial-scale=1.0, width=device-width"/>
            <title>我的Blog</title>
          </Head>
          <Component {...pageProps} />
        </React.Fragment>
      )
    }

    export default MyApp
    ```

  - 局部 head（页面组件）

    ```
    import type { NextPage } from "next"
    import Head from 'next/head'
    import React from "react"

    const List: NextPage = () => {
      return (
        <React.Fragment>
          <Head>
            <meta name="keywords" content="文章列表"/>
            <title>List</title>
          </Head>
          <div>这是List页面</div>
        </React.Fragment>
      )
    }

    export default List
    ```

  为了避免 head 里出现重复的标签，可以给标签加上一个 key 属性：

  ```
  <Head>
    <meta name="keywords" content="文章列表"/>
    <title key="title">List</title>
  </Head>
  <Head>
    <title key="title">new Title</title>
  </Head>
  ```

  这样只会渲染第二个 title

- #### 全局样式

  全局样式只需要在`_app.js`中进行引入即可，在组件中写的样式会变成局部样式，只会影响当前组件。

  Next.js 中有多种方式写局部样式：

  1. CSS Modules

     - 将写好的 SCSS 像模块一样引入到组件中
       ```
       import styles from '../styles/Home.module.scss'
       ```
     - 使用

       ```
       <div className={styles.container}></div>
       ```

       `styles.container`指的就是你提前写好的 container 类名

  2. styled-jsx

     ```
     <div>
       <style jsx>{`
         p {
           color: red;
         }
         span {
           color: green;
         }
       `}</style>
       <p>Hello World <span>啊啊啊啊</span></p>
     </div>
     ```

     总感觉这种会让阅读代码更难....

  3. styled-components

     - 安装：

       ```
       yarn add styled-components -D
       yarn add @types/styled-components -D
       ```

     - 使用：

       ```
       import styled from 'styled-components'
       const App = () => {
         const Title = styled.h1`
           font-size: 24px;
           color: palevioletred;
         `
         return (
           <div>
             <Title>Hello World</Title>
           </div>
         )
       }

       export default Detail
       ```

       这种看起来还不错，不过我还是更喜欢 CSS Modules 的方式

- #### Absolute Imports and Module path aliases

  用过 webpack 都会知道 webpack 中有一个路径别名 alias，Next.js 中也有，而且文档写的很清楚，这里就不赘述了[Absolute Imports and Module path aliases](https://nextjs.org/docs/advanced-features/module-path-aliases)

### 七. 静态资源

Next.js 推荐将静态资源放在 public 中，然后就可以这样用：

`<link rel="icon" href="/favicon.ico" />`

但是放在 public 中的资源不会受构建工具的处理，他会直接把 public 中的资源复制到最终的打包目录下面，所以像网站的 icon 这类基本不会变的资源适合放在 public 中，页面使用的一些经常变的资源就不适合了，原因有：

1. 资源文件名不能修改，会导致资源在客户端的更新不及时
2. 不能使用构建工具进行优化，比如图片压缩

所以那些经常变的资源需要放在其他目录中，然后要自己配置 webpack 进行处理：

1. 新建 assets 文件夹作为这些资源存放的目录
2. 项目根目录新建`next.config.js`

   ```
   module.exports = {
     webpack: (config, options) => {
       // webpack 配置
       return config
     }
   }
   ```

   这是[文档](https://nextjs.org/docs/api-reference/next.config.js/custom-webpack-config)，它这里的配置语法用的应该是[webpack-chain](https://github.com/neutrinojs/webpack-chain)

### 八. Next.js API

这里说的 API 指的就是平时在页面中请求的接口，下面来实现一个获取文章列表的接口：

#### 1. 创建几篇文章

![](/madao.github.io/database/images/articles/node/next/image1.png)

#### 2. 安装 gray-matter 用于将 md 文件解析成对象格式

`yarn add gray-matter -D`

#### 3. 在`pages/api`目录下创建`articles.tsx`

注意 Next.js 中接口都要放在这个 api 目录下

```
import type { NextApiHandler, NextApiRequest, NextApiResponse } from "next"
import fs, { promises as fsPromise } from 'fs'
import path from 'path'
import grayMatter from 'gray-matter'

const getArticles = async () => {
  const articlesPath = path.join(process.cwd(), 'articles')
  const fileNames = await fsPromise.readdir(articlesPath)
  return fileNames.map((name, index) => {
    const content = fs.readFileSync(path.join(articlesPath, name), 'utf-8')
    const { title, date } = grayMatter(content).data
    return {
      id: index,
      title,
      date
    }
  })
}
const articles: NextApiHandler = async (request, response) => {
  response.statusCode = 200
  response.setHeader('Content-Type', 'application/json')
  const articles = await getArticles()
  response.json(articles)
}

export default articles
```

然后访问`http://localhost:1234/api/articles`就会得到如下数据：

![](/madao.github.io/database/images/articles/node/next/image2.png)

这就实现了一个获取文章列表的接口，只不过文章列表是读取的本地文件目录而不是从数据库中获取的

### 九. Next.js 中的三种渲染

在 Next.js 可以使用三种方式对页面进行渲染：

1. 客户端渲染
2. 静态页面生成（SSG）
3. 服务端渲染（SSR）

静态页面生成以及服务端渲染都叫做“预渲染”

#### 1. 客户端渲染

客户端渲染是最常见的一种方式，它的意思就是在浏览器上渲染页面，页面中的动态内容通过接口获取，获取到之后再通过 JS 生成 DOM 进行渲染。

客户端渲染有两个缺点：

- 白屏问题
- 对 SEO 不友好

客户端渲染所需要的数据通过 AJAX 获取，那么在数据没有到来的时候呈现给用户的就是白屏或者是一个 loading，对 SEO 不友好指的是当爬虫爬到页面的时候，他是不会执行 js 去请求数据的，所以爬虫爬到的 html 中没有内容

继续接上面的例子来举例：

List.tsx

```
import type { NextPage } from "next"
import React, { useState, useEffect } from "react"

const List: NextPage = () => {
  const [articles, setArticles] = useState<{
    id: number;
    title: string;
    date: string;
  }[]>([])

  useEffect(() => {
    const xhr = new XMLHttpRequest()
    xhr.onreadystatechange = () => {
      if (xhr.status === 200 && xhr.readyState === 4) {
        setArticles(JSON.parse(xhr.responseText))
      }
    }
    xhr.open('get', '/api/articles')
    xhr.send()
  }, [])
  return (
    <React.Fragment>
      {articles.length ? (
        <ul>
          { articles.map(article => (
            <li key={article.id}>
              <h2>{article.title}</h2>
              <p>日期：{article.date}</p>
            </li>
          )) }
        </ul>
      ): <div>loading....</div>}
    </React.Fragment>
  )
}

export default List
```

这就是一个最简单的客户端渲染的方式，文章列表是通过 AJAX 请求获得的，所以一开始浏览器获取到的 HTML 是：

![](/madao.github.io/database/images/articles/node/next/image3.png)

#### 2. 静态页面生成

试想这样的场景，你的博客在每一个访问者看来内容都是一样的，一般不会出现根据访问者来呈现不同的内容，所以页面的内容没有必要等到客户端发起请求获取，可以在页面生成（构建）的时候直接把文章的内容填充到 html 中去，客户端拿到的 html 就是有内容的，对 SEO 也很友好，而且省下了请求时间，白屏的问题也会被优化。

同样页面静态化也是有缺点的：

1. 无法根据用户身份提供内容，比如微博首页它会根据用户身份提供不同的推荐内容
2. 当接口数据发生改变的时候，需要重新生成页面，因为静态化的时机是在页面构建的时候，接口数据发生变化页面是不知道的。

优点就是上面说的：

1. 缩短白屏时间，优化用户体验
2. 便于 SEO，搜索引擎的爬虫能获取到页面的完整内容

静态化的方式很适合做个人博客

接下来看看如何在 Next.js 中进行页面静态生成

1. 在当前页面组件导出一个`getStaticProps`函数，还是以 List 组件举例：

   ```
   import type { NextPage, GetStaticProps, InferGetStaticPropsType } from "next"
   import React from "react"
   import { getArticles } from 'pages/api/articles'

   const List: NextPage<InferGetStaticPropsType<typeof getStaticProps>> = (props) => {
     return (
       <React.Fragment>
         {props.articles.length ? (
           <ul>
             { props.articles.map(article => (
               <li key={article.id}>
                 <h2>{article.title}</h2>
                 <p>日期：{article.date}</p>
               </li>
             )) }
           </ul>
         ): <div>loading....</div>}
       </React.Fragment>
     )
   }

   export const getStaticProps: GetStaticProps<{
     articles: Array<{ id: number; title: string; date: string; }>
   }> = async () => {
     const articles = await getArticles()
     return {
       props: {
         articles
       }
     }
   }

   export default List
   ```

   之前`getArticles`方法没有导出，所以还要改下`pages/api/articles`里的代码，将`getArticles`导出

2. 执行`yarn build`
3. 执行`yarn start`
4. 访问`http://localhost:3000/List`
5. 同时也可以在`.next/server/pages/List.html`中也可以看到源码，内容是直接填充进去的

#### 3. 服务端渲染

服务端渲染渲染解决了页面静态化的一个问题，就是可以把用户相关的内容静态化，也就是说请求到服务器后，服务器去获取数据，将数据填充到 html 中，直接返回含有完整内容的 html，所以服务端渲染不是在页面构建的时候进行静态化，而是请求到来之后进行的静态化

如何实现呢：

1. getServerSideProps

   例子：

   - List.tsx

     ```
     import type { NextPage, GetServerSideProps, InferGetServerSidePropsType } from "next"
     import React from "react"
     import { getArticles } from 'pages/api/articles'

     const List: NextPage<InferGetServerSidePropsType<typeof getServerSideProps>> = (props) => {
       return (
         <React.Fragment>
           {props.articles.length ? (
             <ul>
               { props.articles.map(article => (
                 <li key={article.id}>
                   <h2>{article.title}</h2>
                   <p>日期：{article.date}</p>
                 </li>
               )) }
               <li>请求参数是：{props.query}</li>
             </ul>
           ): <div>loading....</div>}
         </React.Fragment>
       )
     }

     export const getServerSideProps: GetServerSideProps<{
       articles: Array<{ id: number; title: string; date: string; }>
       query: string;
     }> = async (context) => {
       const articles = await getArticles()
       return {
         props: {
           articles,
           query: JSON.stringify(context.query)
         }
       }
     }

     export default List
     ```

     可以在 getServerSideProps 函数的 context 中获取到请求以及响应的相关信息，可以根据这些信息决定要请求的数据

2. yarn build
3. yarn start
4. 访问`http://localhost:3000/List?hello=123`

服务端渲染的优点：

1. 便于 SEO
2. 缩短白屏时间，优化用户体验
3. 可以将动态内容静态化（根据请求信息来决定页面内容）

服务端渲染的缺点：

1. 无法获取客户端的信息，比如：浏览器窗口大小
2. 将请求和 html 文件动态生成放在了服务端，相当于服务端帮客户端做了这些工作，会加大服务端的压力，但是我还没完整的实现一个应用所以对服务端影响的大小不确定

### 十. 动态路由

还是用上面的例子说明，文章详情其实是同样的页面，只是根据不同的 id 来判断展示那篇文章，这种情况的预渲染可以用动态路由来实现

动态路由需要注意的就是创建组件的时候，文件名要命名成`[id].tsx`格式，`[]`里面不一定是 id，也可以是其他字符，它表示一个占位符，还要组件内需要`export` `getStaticPaths`函数，文档上的例子更清晰，这里就不举例了（主要是公司放假啦！！！！）

[Dynamic Routes](https://nextjs.org/learn/basics/dynamic-routes/dynamic-routes-details)
