每次学习框架之前，我都会去看对应框架的官网对该框架是如何描述的，让人沮丧的是从来没有真正看懂过。所以这次还是一样，直接用，边用边理解吧。

### 一. 项目初始化

```
yarn add express
```

新建 index.js 文件

```

const express = require('express');

const app = express();

app.use('/', (req, res) => {

    res.send('hello express')

})

app.listen(3000);

```

结果：

![](/madao.github.io/database/images/articles/node/express/image.png)

这一部分来看：

`const app = express();`就好像原生 http 模块中的`const server = http.createServer();`

`app.use` 就像是原生 http 模块路径匹配一样。

相比原生模块不用写监听 request 事件

### 二. 中间件

我理解的中间件是，当 express 接受到一个请求时，从接受到返回这中间的过程里，可以使用各种处理方法对这个请求进行处理，处理的这些逻辑（函数）就叫做中间件。举个例子：

```

const express = require('express');

const app = express();

let human = [];

app.use('/human', (req, res, next) => {

  res.message = '打完这场战争，我就要回老家结婚';

  console.log('我来组成头部');

  human.push('head');

  next();

})

app.use('/human', (req, res, next) => {

  console.log('我来组成身体');

  human.push('body');

  next();

})

app.use('/human', (req, res, next) => {

  console.log('我来组成双手');

  human.push('hands');

  next();

})

app.use('/human', (req, res, next) => {

  console.log('我来组成双腿');

  human.push('legs');

  next();

})

app.use('/human', (req, res, next) => {

  console.log('我来组成双脚');

  console.log(res.message)

  human.push('feet');

  res.json(human);

  human = [];

})

app.listen(3000);

```

结果:

![](/madao.github.io/database/images/articles/node/express/image1.png)

这是 log 出来的信息：

![](/madao.github.io/database/images/articles/node/express/image2.png)

可以看出中间件两个特征：

1. 中间件执行顺序和注册顺序，也就是写的顺序是相同的。
2. 对一个请求来说从第一个中间件到最后一个中间件，它们之间的 response 都是共享的，当然 req 也是共享的，如果当前中间件如果修改了 response，那么下一个中间件获取到的 response 就是修改过后的 response
3. 如果不调用 next 方法，那么就不会执行下一个中间件。在例子中没有体现出来，可自行测试。

- app.use

  app.use 专业一点的叫法叫应用级中间件，看了例子不难猜出，它的第一个参数就是要匹配的 url 路径。第二个参数就是处理函数。除了例子中的写法，还可以这样写：

  ```

  const express = require('express');

  const app = express();

  let human = [];

  function middleware1(req, res, next) {

      res.message = '打完这场战争，我就要回老家结婚';

      console.log('我来组成头部');

      human.push('head');

      next();

  }

  function middleware2(req, res, next) {

      console.log('我来组成身体');

      human.push('body');

      next();

  }

  function middleware3(req, res, next) {

      console.log('我来组成双手');

      human.push('hands');

      next();

  }

  function middleware4(req, res, next) {

      console.log('我来组成双腿');

      human.push('legs');

      next();

  }

  function middleware5(req, res, next) {

      console.log('我来组成双脚');

      console.log(res.message)

      human.push('feet');

      res.json(human);

      human = [];

  }

  app.use('/human', middleware1, middleware2, middleware3, middleware4, middleware5);

  // 这样写也是可以的

  // app.use('/human', [middleware1, middleware2, middleware3, middleware4, middleware5]);

  ```

  处理函数有一种特别的写法：

  ```

  function middleware6(reward) {

      return function (req, res, next) {

          console.log(reward)

          human.push(reward);

          res.json(human);

          human = [];

      }

  }

  ```

  把这个加到例子中：

  ```

  const express = require('express');

  const app = express();

  let human = [];

  function middleware1(req, res, next) {

      res.message = '打完这场战争，我就要回老家结婚';

      console.log('我来组成头部');

      human.push('head');

      next();

  }

  function middleware2(req, res, next) {

      console.log('我来组成身体');

      human.push('body');

      next();

  }

  function middleware3(req, res, next) {

      console.log('我来组成双手');

      human.push('hands');

      next();

  }

  function middleware4(req, res, next) {

      console.log('我来组成双腿');

      human.push('legs');

      next();

  }

  function middleware5(req, res, next) {

      console.log('我来组成双脚');

      console.log(res.message)

      human.push('feet');

      next();

  }

  function middleware6(reward) {

      return function (req, res, next) {

          console.log(reward)

          human.push(reward);

          res.json(human);

          human = [];

      }

  }

  app.use('/human', [

      middleware1,

      middleware2,

      middleware3,

      middleware4,

      middleware5,

      middleware6('赐你富二代属性')

  ]);

  app.listen(3000);

  ```

  结果：

  ![](/madao.github.io/database/images/articles/node/express/image3.png)

  app 除了 use 方法，还有 get，post 等，use 就是说不管请求是什么方法，都会执行当前的中间件，如果是 get 那么只有请求是 get 方法时，才会执行当前的中间件。

  还有一点要注意的是，如果不传第一个参数，也是要匹配的路径，那么所以的请求都会匹配到。

- 错误处理中间件

  错误处理中间件有点特别的是，写的时候必须将参数写全，例子：

  ```
  app.use((error, req, res, next) => {
      console.log(error);
      res.status(500).send(error);
  })
  ```

  如果当前中间件是用来处理错误的，那么就要写上`error, req, res, next`这四个参数。

  例子：

  ```

  const express = require('express');

  const app = express();

  app.use('/whatever', (req, res, next) => {

      next('出错啦！！')

  });

  app.use((error, req, res, next) => {

      console.log(error);

      res.status(500).send(error);

  })

  app.listen(3000);

  ```

  当 next 传入一个参数的时候，express 就会认为你的中间件出错了，就会跳去错误处理的中间件，error 参数就是 next 传入的内容，这里有一个例外就是`next('router')`，如果传的是'router'不会跳去错误处理中间件，这个后面再说。

### 三. express.Router

express.Router()可以创建一个 router 对象，它的用法和上面例子中的 app 用法基本差不多，直接通过例子说明：

index.js

```

const express = require('express');

const app = express();

const router = require('./router');

app.use('/', router);

app.use((error, req, res, next) => {

  console.log(error);

  res.status(500).send(error);

})

app.listen(3000);

```

router.js

```

const express = require('express');

const router = express.Router()

router.use('/', (req, res, next) => {

  console.log('你就是龙宫小姐吗？龙宫礼奈小姐？');

  next();

})

router.use('/', (req, res, next) => {

  console.log('我的车里有空调，让我们去那边谈吧。');

  res.send('我的车里有空调，让我们去那边谈吧。')

})

module.exports = router;

```

结果：

![](/madao.github.io/database/images/articles/node/express/image4.png)

log 信息：

![](/madao.github.io/database/images/articles/node/express/image5.png)

可以看到 router 的执行顺序也是按照注册顺序来的。专业一点的叫法叫路由级中间件。

这样做的好处是，有可能 app 里的逻辑过于复杂，那么继续细分为 router 进行处理，让 app 不至于太臃肿。

还有一种好处是这样：

index.js

```

const express = require('express');

const app = express();

const router = require('./router');

app.use('/hello', router);

app.use((error, req, res, next) => {

  console.log(error);

  res.status(500).send(error);

})

app.listen(3000);

```

router.js

```

const express = require('express');

const router = express.Router()

router.use('/first', (req, res, next) => {

  console.log('你就是龙宫小姐吗？龙宫礼奈小姐？');

  res.send('你就是龙宫小姐吗？龙宫礼奈小姐？')

})

router.use('/second', (req, res, next) => {

  console.log('我的车里有空调，让我们去那边谈吧。');

  res.send('我的车里有空调，让我们去那边谈吧。')

})

router.use('/', (req, res, next) => {

  console.log('空调系统运行正常。');

  res.send('空调系统运行正常。')

})

module.exports = router;

```

请求：`http://localhost:3000/hello`，`router.use('/', () => {})`会被执行。

![](/madao.github.io/database/images/articles/node/express/image6.png)

请求：`http://localhost:3000/hello/first`，`router.use('/first', () => {})`会被执行。

![](/madao.github.io/database/images/articles/node/express/image7.png)

请求：`http://localhost:3000/hello/second`，`router.use('/second', () => {})`会被执行。

![](/madao.github.io/database/images/articles/node/express/image8.png)

前面讲的`next('router')`是这样的：

#### 正常情况下：

index.js

```

const express = require('express');

const app = express();

const router = require('./router');

app.use('/', router);

app.use('/', (req, res) => {

  console.log('进入贤者模式');

  res.send('进入贤者模式');

});

app.use((error, req, res, next) => {

  console.log(error);

  res.status(500).send(error);

})

app.listen(3000);

```

router.js

```

const express = require('express');

const router = express.Router()

router.use('/', (req, res, next) => {

  console.log('你就是龙宫小姐吗？龙宫礼奈小姐？');

  next();

})

router.use('/', (req, res, next) => {

  console.log('我的车里有空调，让我们去那边谈吧。');

  next();

})

router.use('/', (req, res, next) => {

  console.log('空调系统运行正常。');

  res.send('空调系统运行正常。');

})

module.exports = router;

```

请求 http://localhost:3000 的 log 的信息：

![](/madao.github.io/database/images/articles/node/express/image9.png)

#### 当使用了 next('router')之后：

router.js

```

const express = require('express');

const router = express.Router()

router.use('/', (req, res, next) => {

  console.log('你就是龙宫小姐吗？龙宫礼奈小姐？');

  next('router');

})

router.use('/', (req, res, next) => {

  console.log('我的车里有空调，让我们去那边谈吧。');

  next();

})

router.use('/', (req, res, next) => {

  console.log('空调系统运行正常。');

  res.send('空调系统运行正常。');

})

module.exports = router;

```

log 信息：

![](/madao.github.io/database/images/articles/node/express/image10.png)

当使用了 next('router')之后，express 会跳过剩下的路由级中间件，直接进入下一个应用级中间件。

我目前学到的就是以上这些了。

如果想要快速生成一个 express 项目，可以使用`express-generator`这个包。

具体参考文档，[官方文档](https://expressjs.com/en/starter/generator.html)
