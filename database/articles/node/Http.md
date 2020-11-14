在 Node.js 中有一个内置模块叫 http，我们可以用这个模块快速的开发一个 http 服务。

### 一、最基础版本

index.js

```

const http = require('http');



const server = http.createServer();



server.on('request', (request, response) => {

  response.statusCode = 200;

  response.end('hello Node.js');

});



server.listen(3000);

```

在终端启动这个服务 `node index.js`

通过 postman 请求 `http://localhost:3000`，或者直接在浏览器地址栏中输入`http://localhost:3000`，你就会看到 hello Node.js 这行字。

### 二. 不同的 url 路径返回不同的内容

```

const http = require('http');



const server = http.createServer();



server.on('request', (request, response) => {

  const { url } = request;

  console.log(url);

  response.statusCode = 200;

  switch (url) {

    case '/hello':

      response.end('Hello, meet again.');

      break;

    case '/bye':

      response.end('Talk to you next time');

      break;

    default:

      response.end("I don't know what you say.");

      break;

  }

});



server.listen(3000);

```

效果：

![](/madao.github.io/database/images/articles/node/http/image.png)

![](/madao.github.io/database/images/articles/node/http/image1.png)

![](/madao.github.io/database/images/articles/node/http/image2.png)

通过 request 中的 url，就可以根据路径的不同返回不同的内容，注意 request.url 会匹配到域名后的所有内容。

### 三. 处理 get 请求的参数

```

const http = require('http');

const urlParse = require('url');

const qs = require('querystring');



const server = http.createServer();



server.on('request', (request, response) => {

  const { url } = request;

  const { query, pathname } = urlParse.parse(url);

  const { amount } = qs.parse(query);

  response.statusCode = 200;



  switch (pathname) {

    case '/borrowMoney':

      if (Number(amount) > 500) {

        response.end('go away');

      } else {

        response.end('here you are');

      }

      break;

    case '/bye':

      response.end('Talk to you next time');

      break;

    default:

      response.end("I don't know what you say.");

      break;

  }

});



server.listen(3000);



```

通过 url 这个模块，可以将请求的 url 分解成这样一个对象：

```

Url {

  protocol: null,

  slashes: null,

  auth: null,

  host: null,

  port: null,

  hostname: null,

  hash: null,

  search: '?amount=100',

  query: 'amount=100',

  pathname: '/borrowMoney',

  path: '/borrowMoney?amount=100',

  href: '/borrowMoney?amount=100'

}

```

然后通过 querystring 再将分解后的 url 对象中的 query 用`=`分割，这样就可以处理 get 请求带的参数了。

### 四. 处理不同的请求方法

HTTP 请求有很多方法，常见有：GET，POST，PUT，DELETE，OPTIONS，PATCH 等，在 restful 风格下的接口设计，会是同一个 url，通过方法的不同来判断你的这个请求要做什么，比如：

```
http://localhost:3000/user
```

就只有这样一个接口，用 POST 请求新建用户，GET 请求读取用户，PATCH 请求更新用户，DELETE 请求删除用户。

[RESTful API 最佳实践](http://www.ruanyifeng.com/blog/2018/10/restful-api-best-practices.html)

要实现 RESTful 风格的接口，服务器就要进行请求方法的判断。

```

const http = require('http');

const urlParse = require('url');

const qs = require('querystring');



const server = http.createServer();

let users = [];

let userId = 0;



function updateUsers(res, method, id) {

  switch (method) {

    case 'GET':

      res.end(JSON.stringify(users));

      break;

    case 'POST':

      userId += 1;

      const user = { id: userId };

      users.push(user);

      res.end(JSON.stringify(user));

      break;

    default:

      res.statusCode = 405;

      res.end('Method Not Allowed');

      break;

  }

  users = users.filter(item => item.id !== id);

  res.end(JSON.stringify(users));

}

server.on('request', (request, response) => {

  const { url, method } = request;

  const { pathname, query } = urlParse.parse(url);

  const { id } = qs.parse(query);

  response.statusCode = 200;

  switch (pathname) {

    case '/user':

      updateUsers(response, method, id);

      break;

    default:

      response.end("I don't know what you say.");

      break;

  }

});



server.listen(3000);



```

通过 request 获取到请求的方法，然后再去根据方法做相应的处理。

### 五. 处理请求体中的数据

上面的例子中请求的数据都是通过 url 的 query 这种方式发送的，但是实际情况下像 post 请求一般会把请求数据放在请求体中，接下来就看看如何处理请求体中的数据。

看看 postman 上的 body 一栏：

![](/madao.github.io/database/images/articles/node/http/image3.png)

可以看到请求体中的数据类型还是很多的，这些类型通过请求头中的`Content-Type`来决定。

```

const http = require('http');

const urlParse = require('url');



const server = http.createServer();

const users = [];

server.on('request', (request, response) => {

  const { url, method, headers } = request;

  const { pathname } = urlParse.parse(url);

  const contentType = headers['content-type'];

  response.statusCode = 200;

  switch (pathname) {

    case '/user':

      switch (method) {

        case 'GET':

          response.end(JSON.stringify(users));

          break;

        case 'POST':

          if (contentType !== 'application/json') {

            response.statusCode = 400;

            response.end('error');

            break;

          }

          let requestBodyStr = '';

          request.on('data', (data) => {

            requestBodyStr += data.toString();

          });

          request.on('end', () => {

            const user = JSON.parse(requestBodyStr);

            users.push(user);

            response.end(JSON.stringify(user));

          });

          break;

        default:

          response.statusCode = 405;

          response.end('Method Not Allowed');

          break;

      }

      break;

    default:

      response.end("I don't know what you say.");

      break;

  }

});



server.listen(3000);



```

Content-Type 可以从 request 的 headers 中获取。

比较关键的就是这里：

```

let requestBodyStr = '';

request.on('data', (data) => {

    requestBodyStr += data.toString();

});

request.on('end', () => {

    const user = JSON.parse(requestBodyStr);

    users.push(user);

    response.end(JSON.stringify(user));

});

```

要处理请求体中的数据，得让 request 去监听 data 和 end 事件。

这样做的原因是，有可能请求中的数据量很大，比上传一个文件，如果你一次性将数据放在存进内存，那么成百上千个请求发送过来内存就爆了。为了解决这个问题 node.js 使用了一种叫“流”的数据结构来处理这种情况。打个比方：现在有一池塘水，一次性挑完是不现实的，只有一桶一桶的去挑，当你挑的次数足够多的时候那么池塘的水就会被挑完。

data 事件表示流的数据已经可以读取了

end 事件表示这个流已经到末尾了，没有数据可以读取了。

除了这两个之外还有一个 error 事件，表示出错。

如果一桶挑不完，data 事件会被触发多次的。end 事件只会触发一次

做个试验：

```

const http = require('http');

const urlParse = require('url');



const server = http.createServer();

server.on('request', (request, response) => {

  const { url } = request;

  const { pathname } = urlParse.parse(url);

  response.statusCode = 200;

  switch (pathname) {

    case '/upload':

      let count = 0;

      request.on('data', () => {

        count += 1;

      });

      request.on('end', () => {

        response.end(`${count}`);

      });

      break;

    default:

      response.end("I don't know what you say.");

      break;

  }

});



server.listen(3000);



```

![](/madao.github.io/database/images/articles/node/http/image4.png)

通过 form-data 格式传一个 rar 格式的文件。这个文件大小是 65.2MB

结果

![](/madao.github.io/database/images/articles/node/http/image5.png)

node.js 用了 1354 次将池塘里的水挑完。

当都实现了上面的功能后，一个简易的 http 服务就写好了。
