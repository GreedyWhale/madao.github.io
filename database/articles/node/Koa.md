今天学习Koa框架，这个的框架又是TJ大神写的，Express也是他搞得，人和人之间的差距啊🙃。

Koa的文档很简洁，大部分用法和Express差不多，主要的区别:

中间件模型的不同

  在上一篇笔记中有提到Express的中间件模型是线型的：

  ![](/madao.github.io/database/images/articles/node/express2/image1.png)

  而Koa的模型是U型的：

  ![](/madao.github.io/database/images/articles/node/koa/image2.png)

  例子：

  ```
  const koa = require('koa')
  const app = new koa()

  app.use(async (ctx, next) => {
    console.log('1-0')
    await next()
    console.log('1-1')
  })

  app.use(async (ctx, next) => {
    console.log('2-0')
    await next()
    console.log('2-1')
  })

  app.use(async (ctx) => {
    console.log('3-0')
    ctx.body = 'Hello World'
  })

  app.listen(8888, () => {
    console.log('启动成功')
  })
  ```

  输出结果是：

  ```
  1-0
  2-0
  3-0
  2-1
  1-1
  ```

  简单说就是中间件await之前的是中间件第一次执行的代码，await之后是第二次执行的代码

