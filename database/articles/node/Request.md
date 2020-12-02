上一篇总结了 HTTP 模块中响应 HTTP 请求相关的一些知识点，这次总结一下如何主动的发起一个 HTTP 请求，这次通过实现一个翻译的命令行工具来实现。

### 1. 初始化项目

这里就轻车熟路了，使用`yarn init -y` 或者 `npm init -y`生成 package.json，然后根据自己的需求进行修改。

需要安装的依赖：

1. ts-node-dev：上次介绍过，支持 TypeScript 和热更新的 Node 环境
2. commander：一个 生成 Node.js 命令行工具的包
3. typescript

### 2. 找到免费的翻译接口

1. [百度翻译](https://fanyi-api.baidu.com/product/11)

有道好像已经不能白嫖了...

建议选择一些大公司提供的接口，因为使用这些需要上传个人信息，虽然也不能保证大公司不泄露个人信息。

### 3. src/cli.ts

```
import { program } from 'commander'

program
  .version('1.0.0')
  .name('t')
  .usage('<English>')
  .arguments('<English>')
  .action(english => {
    console.log(english)
  })

program.parse(process.argv);
```

在 package.json 中添加

```
"scripts": {
 "t": "tsnd src/cli.ts"
}
```

在终端中输入：

`yarn t -h`

![](/madao.github.io/database/images/articles/node/request/image.png)

`yarn t hello`

![](/madao.github.io/database/images/articles/node/request/image1.png)

现在可以正常的拿到输入的英文单词了。

小知识点：如果文档上的参数是`<>`抱起来就的表示这个参数是必传参数，`[]`包起来的表示是可选参数

### 3. src/main.ts

HTTP 模块中用于发出请求的方法是：`http.request`，如果是 get 请求可以使用`http.get`，这里采用`http.request`进行演示：

- 测试请求是否可以正常发出

  ```
  import * as http from 'https'

  const request = http.request({
    host: 'www.baidu.com',
    path: '/',
    port: 443,
    method: 'GET'
  }, (response) => {
    let chunks = ''
    response.on('data', (chunk: Buffer) => {
      chunks += chunk.toString()
    })
    response.on('end', () => {
      console.log(chunks)
    })
  })

  request.on('error', (error) => {
    console.error(error.message)
  })

  request.end()
  ```

  - 根据请求的地址使用的协议决定使用的模块，比如 http 协议就用 http 模块，https 协议就使用 https 模块
  - http.request 方法需要调用 request.end()来发出请求
  - GET 请求的参数携带在 path 中，POST 请求的参数需要使用 request.write 进行写入
  - 在使用 TypeScript 的过程中，如果你不知道某个数据的类型，可以通过打印该数据的 constructor 来得到，比如代码中的 chunk，在 Node.js 的类型文件中写的是 any，这时候就可以通过：`console.log(chunk.constructor)`得到 chunk 的真正的类型：Buffer

  在终端中输入：`yarn tsnd src/main.ts`，就可以得到如下结果：

  ![](/madao.github.io/database/images/articles/node/request/image2.png)

  成功的请求到了 baidu 首页的 HTML

- 根据翻译接口文档实现翻译功能

  有道提供的接口好像不免费了，所以这里选择百度提供的免费翻译接口，去官网注册账号后，可以拿到一个 APP ID 和密钥，这个需要保存好，不要公布出去

  ```
  import * as http from 'https'
  import * as querystring from 'querystring'
  import { appID, secretKey } from './private'

  const md5 = require('md5')
  const getParams = (word: string) => {
    const salt = Math.random()
    const sign = md5(`${appID}${word}${salt}${secretKey}`)
    const params = querystring.stringify({
      q: word,
      from: 'en',
      to: 'zh',
      appid: appID,
      salt,
      sign
    })
    return params
  }
  const translation = (word: string) => {
    const params = getParams(word)
    const request = http.request({
      host: 'fanyi-api.baidu.com',
      path: `/api/trans/vip/translate?${params}`,
      port: 443,
      method: 'GET'
    }, (response) => {
      let chunks = ''
      response.on('data', (chunk: Buffer) => {
        chunks += chunk.toString()
      })
      response.on('end', () => {
        console.log(chunks)
      })
    })

    request.on('error', (error) => {
      console.error(error.message)
    })

    request.end()
  }

  export default translation
  ```

  - 从上面代码可以看出，我将 appid 和密钥存在了 private 这个文件中，然后需要创建一个`.gitignore`文件将这个文件的路径写进去，让 git 不要上传这个文件，但是如果要发布 npm 包，那么 appid 和密钥这个还是会暴露出去了，请务必小心。

  - 由于文档上 sign 需要 md5 进行转换，所以需要下载 md5 及@types/md5，md5 这个包只支持 commonjs 的模块，所以只能使用 require 进行引入。

- src/cli.ts

  ```
  import { program } from 'commander'
  import translation from './main'

  program
    .version('1.0.0')
    .name('t')
    .usage('<English>')
    .arguments('<English>')
    .action(translation)

  program.parse(process.argv);
  ```

在终端执行`yarn t hello`得到：

![](/madao.github.io/database/images/articles/node/request/image3.png)

dst 字段就是翻译的结果，但是中文显示的是 unicode 编码接下来进行优化。

### 4. 优化

- 友好的 loading 提示

  翻译结果是通过接口获取到的，请求接口是要时间的，在这期间给用户一个 loading 的提示比空白要好很多，命令行中 loading 可以使用[ora](https://www.npmjs.com/package/ora)这个工具实现：

  src/main.ts

  ```
  import * as http from 'https'
  import * as querystring from 'querystring'
  import { appID, secretKey } from './private'

  const md5 = require('md5')
  const ora = require('ora')
  const getParams = (word: string) => {
    const salt = Math.random()
    const sign = md5(`${appID}${word}${salt}${secretKey}`)
    const params = querystring.stringify({
      q: word,
      from: 'en',
      to: 'zh',
      appid: appID,
      salt,
      sign
    })
    return params
  }
  const getErrorMessages = (errorCode) => {
    const errorMessage = {
      52001: ['请求超时','重试'],
      52002: ['系统错误','重试'],
      52003: ['未授权用户', '请检查您的appid是否正确，或者服务是否开通'],
      54000: ['必填参数为空','请检查是否少传参数'],
      54001: ['签名错误', '请检查您的签名生成方法'],
      54003: ['访问频率受限', '请降低您的调用频率，或进行身份认证后切换为高级版/尊享版'],
      54004: ['账户余额不足', '请前往管理控制台为账户充值'],
      54005: ['长query请求频繁', '请降低长query的发送频率，3s后再试'],
      58000: ['客户端IP非法','检查个人资料里填写的IP地址是否正确，可前往开发者信息-基本信息修改，可前往开发者信息-基本信息修改'],
      58001: ['译文语言方向不支持', '检查译文语言是否在语言列表里'],
      58002: ['服务当前已关闭', '请前往管理控制台开启服务'],
      90107: ['认证未通过或未生效', '请前往我的认证查看认证进度']
    }
    return errorMessage[errorCode] || ['出错了', '请稍后重试']
  }
  const loading = ora()
  const translation = (word: string) => {
    loading.start('loading')
    const params = getParams(word)
    const request = http.request({
      host: 'fanyi-api.baidu.com',
      path: `/api/trans/vip/translate?${params}`,
      port: 443,
      method: 'GET'
    }, (response) => {
      let chunks = ''
      response.on('data', (chunk: Buffer) => {
        chunks += chunk.toString()
      })
      response.on('end', () => {
        const result = JSON.parse(chunks)
        if (result.error_code) {
          const errorMessages = getErrorMessages(result.error_code)
          loading.fail(`${errorMessages[0]}`)
          loading.info(`${errorMessages[1]}`)
        } else {
          loading.succeed(`${result.trans_result[0].dst}`)
        }
        process.exit(0)
      })
    })

    request.on('error', (error) => {
      loading.fail(error.message)
      process.exit(0)
    })

    request.end()
  }

  export default translation
  ```

  效果：

  ![](/madao.github.io/database/images/articles/node/request/image4.png)
  ![](/madao.github.io/database/images/articles/node/request/image5.png)
  ![](/madao.github.io/database/images/articles/node/request/image6.png)

### 5. 发布

这里说下如何发布 ts 写的包：

1. 全局或者在项目中安装 Typescript，然后在终端中执行

   ```
   // 项目中安装
   yarn tsc --init
   ```

   ```
   // 全局安装
   tsc --init
   ```

   这个命令会在项目中生成一个 tsconfig.json 文件

2. 修改 tsconfig.json

   将 tsconfig.json 中的 outDir 字段修改为:

   ```
   "outDir": "./dist"
   ```

   然后执行：tsc，会得到一个 dist 目录，里面就是 ts 转为 js 后的文件：

   ![](/madao.github.io/database/images/articles/node/request/image7.png)

   然后修改 package.json 中的 main 和 file 字段：

   ```
   "main": "dist/cli.js",
   "files": [
     "dist"
   ]
   ```

   然后执行：`npm login` 登录你的 npm 账户，最后执行`npm publish`进行发布

3. 修改 TypeScript 告诉你的错误

   当项目存在 tsconfig.json 后，TypeScript 按照配置文件中的配置去检测代码，类似这样：

   ![](/madao.github.io/database/images/articles/node/request/image8.png)

   需要消除这些红线后再发布

[源码](https://github.com/GreedyWhale/node-fy-cli)
