### 一. 框架

选用 express 框架，[文档](https://expressjs.com/)

```
npm init

npm install express --save
```

### 二. 简单测试请求

在当前目录新建 index.js 文件

```

const express = require("express");



const app = express();



app.get("/", (req, res) => {

    res.send("Hello Node.js");

});



const port = 3000;

app.listen(port);

```

在终端输入：
`node index.js`

在浏览器中打开`127.0.0.1:3000`

![](/caisr.github.io/database/images/articles/gadgets/web_server/image.png)

### 三.使用 form 上传图片

```

<!DOCTYPE html>

<html>

<head>

  <meta charset="utf-8">

  <meta name="viewport" content="width=device-width">

  <title>upload</title>

</head>

<body>

  <form action="http://127.0.0.1:3000/upload" method="post" enctype="multipart/form-data">

    <div>

      <input type="file" name="avatar" accept="image/*">

    </div>

    <input type="submit" value="上传">

  </form>

</body>

</html>

```

![](/caisr.github.io/database/images/articles/gadgets/web_server/image1.png)

将 index.js 中的接口更新成

```
app.post("/upload", (req, res) => {
  res.send('上传成功')
});
```

注意：index.js 中的文件只要改了，就要重新启动服务

试着上传一下：

![](/caisr.github.io/database/images/articles/gadgets/web_server/image2.png)

### 四. 将前端发送的图片储存在服务器中

这里需要用到一个叫[multer](https://github.com/expressjs/multer)的库

`npm install multer --save`

根据他的文档，改一下 index.js:

```

const express = require("express");

const multer = require("multer");



// 这里定义图片储存的路径，是以当前文件为基本目录

const upload = multer({ dest: "uploads/" });

const app = express();

/*

  upload.single('avatar') 接受以avatar命名的文件，也就是input中的name属性的值

  avatar这个文件的信息可以冲req.file中获取

*/

app.post("/upload", upload.single("avatar"), (req, res) => {

  console.log(req.file);

  res.send("上传成功");

});



const port = 3000;

app.listen(port);

```

改完之后重新启动服务，再重新上传：

![](/caisr.github.io/database/images/articles/gadgets/web_server/image3.png)

可以看到 req.file 中就是上传的文件信息。

同时，你会发现当前目录下，会多一个文件夹叫 uoloads。

![](/caisr.github.io/database/images/articles/gadgets/web_server/image4.png)

那个很长名字的文件，就是刚刚前端传的图片。只要改一下后缀名就可以预览了：

![](/caisr.github.io/database/images/articles/gadgets/web_server/image5.png)

### 五. 将储存的图片名返回给前端

一般上传完头像会有一个预览功能，那么只需要后端将上传后的图片名发送给前端，前端重新请求一下图片就好了，前面都是用 form 默认的提交，这个提交存在一个问题就是，提交完成后页面会发生跳转。所以现在一般都是用 ajax 进行上传。

```

// js代码

upload.addEventListener('submit', (e) => {

  // 阻止form 的默认行为

  e.preventDefault();



  // 创建FormData对象

  var formData = new FormData();

  var fileInput = document.querySelector('input[name="avatar"]');

  formData.append(fileInput.name, fileInput.files[0]);



  // 创建XMLHttpRequest

  var xhr = new XMLHttpRequest();

  xhr.open('POST', upload.action);

  xhr.onload = function() {

    console.log(xhr.response)

  }

  xhr.send(formData);

})

```

- 小提示：如果 HTML 的元素有 id 属性，那么可以不用`document.querySelector`去选中它，可以直接使用，就像全局变量一样。

```

// index.js

app.post("/upload", upload.single("avatar"), (req, res) => {

  res.json({name: req.file.filename }); // 使用json格式返回数据。

});



```

这时候重新发送，会出现一个问题：

![](/caisr.github.io/database/images/articles/gadgets/web_server/image6.png)

由于代码是写在 JS Bin 上的，使用 AJAX 请求不同域名的接口，会出现跨域情况，解决这个问题需要，在 index.js 中加上一个头部，就是报错信息中的`Access-Control-Allow-Origin`：


```

app.post("/upload", upload.single("avatar"), (req, res) => {

  res.set('Access-Control-Allow-Origin', '*');

  res.json({name: req.file.filename });

});

```

- `*`表示所有其他域名都可访问，也可以将`*`改为其他允许的域名。

重新发送：

![](/caisr.github.io/database/images/articles/gadgets/web_server/image7.png)

这样成功的上传了图片，并且拿到了上传后的图片名。

这里可以使用一个库[cors](https://www.npmjs.com/package/cors)，来完成添加响应头的操作：

`npm install cors --save`

修改 index.js

```

const express = require("express");

const multer = require("multer");

const cors = require('cors');  // 新增

// 这里定义图片储存的路径，是以当前文件为基本目录

const upload = multer({ dest: "uploads/" });

const app = express();



app.use(cors());  // 新增

/*

  upload.single('avatar') 接受以avatar命名的文件，也就是input中的name属性的值

  avatar这个文件的信息可以冲req.file中获取

*/

app.post("/upload", upload.single("avatar"), (req, res) => {

  res.json({name: req.file.filename });

});



const port = 3000;

app.listen(port);



```

### 五. 展示上传后的图片

新定义一个接口：

```

app.get("/preview/:name", (req, res) => {

  res.sendFile(`uploads/${req.params.name}`, {

    root: __dirname,

    headers:{

      'Content-Type': 'image/jpeg',

    },

  }, (error)=>{

    if(error){

      res.status(404).send('Not found')

    }

  });

});

```

`/preview:name`这种方式定义接口的路径，请求过来的时候，就可以从 req.params.name 中拿到`/preview/xxxx`中的 xxxx 了。

修改下 HTML:

```

<!DOCTYPE html>

<html>

<head>

  <meta charset="utf-8">

  <meta name="viewport" content="width=device-width">

  <title>JS Bin</title>

</head>

<body>

  <form id="upload" action="http://127.0.0.1:3000/upload" method="post" enctype="multipart/form-data">

    <div>

      <input type="file" name="avatar" accept="image/*">

    </div>

    <input type="submit" value="上传">

  </form>

  <img src="" id="avatarImg">  <!-- 新增 -->

</body>

</html>

```

改下 js

```

upload.addEventListener('submit', (e) => {

  // 阻止form 的默认行为

  e.preventDefault();



  // 创建FormData对象

  var formData = new FormData();

  var fileInput = document.querySelector('input[name="avatar"]');

  formData.append(fileInput.name, fileInput.files[0]);



  // 创建XMLHttpRequest

  var xhr = new XMLHttpRequest();

  xhr.open('POST', upload.action);

  xhr.onload = function() {

    var imgName = JSON.parse(xhr.response).name;  // 新增

    avatarImg.setAttribute('src', 'http://127.0.0.1:3000/preview/' + imgName); // 新增

  }

  xhr.send(formData);

})

```

结果：

![](/caisr.github.io/database/images/articles/gadgets/web_server/image8.png)

### 六. 将代码部署到 Heroku

Heroku 是一个支持多种编程语言的云平台即服务。最重要的它是免费的。这是他的官方网站[Heroku](https://www.heroku.com/)，注意不科学上网的话，可会超级慢或者进不去。而且科学上网要全局模式..

1. 注册，过程省略
2. 选择创建一个新的应用

    ![](/caisr.github.io/database/images/articles/gadgets/web_server/image9.png)

3. 在部署的时候，有三个选择，我选择选择 GitHub
4. 由于选择 GitHub，那么还需要创建一个仓库，把代码放上去。
5. 放上去之间还要改一下代码：

   因为部署是交给 heroku 的，所以端口号不能写死：

   ```
   const port = process.env.PORT || 3000;
   ```

6. 在 package.json 添加一个 npm start 命令

   `"start": "node index.js"`

7. 在 GitHub 上创建仓库并上传代码，过程略,别忘了写.gitignore 文件
8. 这是我的[仓库地址](https://github.com/GreedyWhale/node-img-server)
9. 在 heroku 中选择仓库并且选择分支 master，部署

    ![](/caisr.github.io/database/images/articles/gadgets/web_server/image10.png)

10. 预览

    ![](/caisr.github.io/database/images/articles/gadgets/web_server/image11.png)


    这个就是部署好的域名了。

用这个域名试一试

![](/caisr.github.io/database/images/articles/gadgets/web_server/image12.png)

大功告成。可惜的就是 heroku 得科学上网才行。
