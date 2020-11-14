### Getting Started Example

继续跟着官方文档学习

### 一. 项目搭建

这次官方文档上的这个项目是一个可以改变页面背景色的扩展程序。

1. 新建 GettingStartedExample 文件夹，作为项目目录。
2. 创建 manifest.json
   ```
   {
     "manifest_version": 2,
     "name": "Getting Started Example",
     "version": "1.0",
     "description": "Build an Extension!"
   }
   ```

### 二. 完善项目

#### 1. background

manifest.json 中有一个字段叫 background，background 包含两个属性：

```
"background": {
  "scripts": ["background.js"],
  "persistent": false
},
```

在 background 的 scripts 属性中的 js 文件会常驻后台，注意 scripts 属性是一个数组，意味着可以让多个 js 文件常驻后台。举个例子：

在当前项目新建 js 文件夹，并创建 background.js:

```
setInterval(() => {
  console.log(new Date().toLocaleString())
}, 1000)
```

在 manifest.json 中添加：

```
"background": {
  "scripts": ["js/background.js"],
  "persistent": false
}
```

在开发模式下安装这个扩展程序：

![](/madao.github.io/database/images/articles/google/getting_started_example/image.png)

点击`背景页`按钮，会弹出如下界面：

![](/madao.github.io/database/images/articles/google/getting_started_example/image1.png)

background.js 的代码会一直在后台执行。

除了 scripts 属性，还有一个属性是 persistent，它定义了常驻后台的方式：

- true： 一直在后台运行，从浏览器打开到浏览器关闭期间，js 文件一直运行
- false：按需运行，只在需要时运行，比如触发了一些事件，扩展程序第一次安装或更新新版本，这是官网的描述，英语太差就不翻译了：
  > A background page is loaded when it is needed, and unloaded when it goes idle. Some examples of events include:
  >
  > - The extension is first installed or updated to a - new version.
  > - The background page was listening for an event, and the event is dispatched.
  > - A content script or other extension sends a message.
  > - Another view in the extension, such as a popup, calls runtime.getBackgroundPage.

例子中配置了`persistent: false`，那么 background.js 就是按
需运行，在扩展程序管理页面，会发现过一段时间后就会出现这样的情况：

![](/madao.github.io/database/images/articles/google/getting_started_example/image2.png)

表示后台脚本终止了运行。

还可以通过 Chrome 的任务管理器，查看后台脚本的生命周期：

![](/madao.github.io/database/images/articles/google/getting_started_example/image3.png)

几秒钟不活动后，后台脚本会自行卸载。释放占用的资源。
`persistent: false` 是 Chrome 官方推荐的方式

现在删除测试代码，按照官方文档将 background.js 改为：

```
  chrome.runtime.onInstalled.addListener(function() {
    chrome.storage.sync.set({color: '#3aa757'}, function() {
      console.log("The color is green.");
    });
});
```

解释下这个代码：
监听扩展程序第一次安装、更新或是 Chrome 更新事件，事件的回调中在本地存储一个`{color: '#3aa757'}`对象。

Chrome 提供的本地储存和 Window.localStorage 是有些不同的，
具体可参考：
[[14] chrome 扩展开发 - chrome.storage 本地存储](http://www.ptbird.cn/chrome-extensions-storage.html)

这里是[官方文档](https://developer.chrome.com/extensions/storage)

接下来还需要在 manifest.json 中添加以下属性：

```
"permissions": ["storage"],
```

扩展程序调用某些 Chrome 提供的 API 需要申请才行，permissions 就是用于权限申请的字段。

#### 2. 添加操作界面

创建 popup.html：

```
<!DOCTYPE html>
<html>
  <head>
    <style>
      button {
        height: 30px;
        width: 30px;
        outline: none;
      }
    </style>
  </head>
  <body>
    <button id="changeColor"></button>
  </body>
</html>
```

需要注意的是：在 Chrome 扩展程序中的 HTML 中，JavaScript 代码必须使用外部引用的方式，就是使用这种方式：

```html
<script src="xx.js"></script>
```

在 manifest.json 中添加：

```
"page_action": {
  "default_popup": "popup.html",
  "default_icon": {
    "16": "images/logo16.png",
    "32": "images/logo32.png",
    "48": "images/logo48.png",
    "128": "images/logo128.png"
  }
},
"icons": {
  "16": "images/logo16.png",
  "32": "images/logo32.png",
  "48": "images/logo48.png",
  "128": "images/logo128.png"
}
```

- page_action
  之前有提到过，有 page_action 字段的扩展程序，不是所有页面都可用的，需要页面满足一些特定条件才可用。page_action 字段同样含有这三个属性：

  ```
  "page_action": {
    "default_icon": {
      "16": "images/icon16.png",
      "24": "images/icon24.png",
      "32": "images/icon32.png"
    },
    "default_title": "Google Mail",
    "default_popup": "popup.html"
  },
  ```

  - default_icon：展示在工具栏上的图标路径
  - default_title：鼠标悬浮在工具栏上的图标时显示的文字
  - default_popup：点击工具栏上的图标时展示的页面

- icons

  icons 定义的是扩展程序的图标，这个图标会在 Chrome 的扩展商店、扩展程序管理页面、工具栏上展示（如果你没有配置 page_action 或 browser_action 中的 default_icon 的话），例子：

  ![](/madao.github.io/database/images/articles/google/getting_started_example/image4.png)

  官方推荐开发者应始终提供 128 * 128 和 48*48 的图标，图标最好是正方形的。

#### 3. 设置扩展程序生效的条件

上一步使用了 page_action 这个字段，如果现在安装例子中的项目会发现：

![](/madao.github.io/database/images/articles/google/getting_started_example/image5.png)

图标是暗的，而且点击没有任何反应，现在就需要设置一些条件，当页面满足这些条件的时候，扩展程序就可以用了：

background.js

```diff
chrome.runtime.onInstalled.addListener(function() {
  chrome.storage.sync.set({color: '#3aa757'}, function() {
    console.log("The color is green.");
  });

+  chrome.declarativeContent.onPageChanged.removeRules(undefined, function() {
+    chrome.declarativeContent.onPageChanged.addRules([{
+      conditions: [new chrome.declarativeContent.PageStateMatcher({
+        pageUrl: {hostEquals: 'developer.chrome.com'},
+      })],
+      actions: [new chrome.declarativeContent.ShowPageAction()]
+    }]);
+  });
});
```

- chrome.declarativeContent.PageStateMatcher：这个 API 可以让开发者根据页面的 URL 或是内容匹配的 CSS 选择器进行某些操作，比如：

  ```
  conditions: [
    new chrome.declarativeContent.PageStateMatcher({
      pageUrl: { hostEquals: 'www.google.com', schemes: ['https'] },
      css: ["input[type='password']"]
    })
  ],
  actions: [ new chrome.declarativeContent.ShowPageAction() ]
  ```

  上面代码中规定的条件就是当页面满足：协议为 https，域名为www.google.com，而且页面上有能被`input[type='password']`这个css选择器选中的元素（页面含有密码类型的输入框），那么就会激活当前扩展程序。

- chrome.declarativeContent.onPageChanged.removeRules： 删除规则，第一个参数是 undefined 表示删除扩展程序的所有规则。
- chrome.declarativeContent.onPageChanged.addRules：添加规则。
- new chrome.declarativeContent.ShowPageAction：激活扩展程序

看例子中的代码，有一处写的很奇怪，就是先删除了所有的规则，然后才添加新的规则，查找文档发现这么一段话：

> Note: Rules are persistent across browsing sessions. Therefore, you should install rules during extension installation time using the runtime.onInstalled event. Note that this event is also triggered when an extension is updated. Therefore, you should first clear previously installed rules and then register new rules.

大意是扩展程序添加的规则是持久存在的，所以规则只需要在扩展程序安装的时候添加一次就可以，在 runtime.onInstalled 的回调中添加规则，但是这个事件在扩展程序更新、Chrome 更新时都会触发，为了避免重复添加，所以需要先清除之前添加的规则。

declarativeContent 这 API 也需要申请才能用，所以在 manifest.json 中添加

```diff
"permissions": [
+  "declarativeContent",
    "storage"
]
```

这时候在 Chrome 中重新刷新例子中的扩展程序（如果不生效尝试删除后重新安装），就可以看到：

![](/madao.github.io/database/images/articles/google/getting_started_example/image6.png)

在满足这些条件的页面上，例子中的扩展程序就可以用了。

#### 4. 完善扩展程序功能

创建 js/popup.js 文件

```javascript
chrome.storage.sync.get("color", function (data) {
  changeColor.style.backgroundColor = data.color;
  changeColor.setAttribute("value", data.color);
});
```

在 popup.html 中引入:

```diff
<body>
   <button id="changeColor"></button>
+  <script src="js/popup.js"></script>
</body>
```

由于 button 元素有 id，所以它变成了一个全局变量。直接通过 changeColor 就可以获取到。

刷新扩展程序，可以看到：

![](/madao.github.io/database/images/articles/google/getting_started_example/image7.png)

按钮的颜色变成了在 background.js 中存的颜色了

接下来在 popup.js 中添加：

```
changeColor.onclick = ({ target: { value } }) => {
  chrome.tabs.query({active: true, currentWindow: true}, (tabs) => {
    chrome.tabs.executeScript(
      tabs[0].id,
      {code: `document.body.style.backgroundColor = "${value}";`});
  });
};
```

这段代码的意思就是在 Chrome 的选项卡中找到当前窗口中激活的选项卡，然后执行

```javascript
`document.body.style.backgroundColor = "${value}";`;
```

这段代码。

要想用 chrome.tabs.executeScript 这个 API，要去申请权限，在 manifest.json 中添加：

```diff
"permissions": [
+  "activeTab",
   "storage",
   "declarativeContent"
]
```

再去 Chrome 扩展程序管理页面刷新扩展程序，这时候就可以改变页面的背景色了：

![](/madao.github.io/database/images/articles/google/getting_started_example/image8.png)

#### 5. 添加选项页面

选项页面就是提供给用户做一些其他配置的，看看其他扩展程序的选项页是什么样的：

![](/madao.github.io/database/images/articles/google/getting_started_example/image9.png)


![](/madao.github.io/database/images/articles/google/getting_started_example/image10.png)

从截图中可以看到 JSON Viewer 这个扩展程序，在他的选项页提供了一些主题供用户选择，要实现这个功能需要用到 options_page 这个字段。

给 manifest.json 中添加：

```
"options_page": "options.html"
```

创建 options.html

```html
<!DOCTYPE html>
<html>
  <head>
    <style>
      button {
        height: 30px;
        width: 30px;
        outline: none;
        margin: 10px;
      }
    </style>
  </head>
  <body>
    <div id="buttonDiv"></div>
    <div>
      <p>Choose a different background color!</p>
    </div>
  </body>
  <script src="js/options.js"></script>
</html>
```

创建 js/options.js

```javascript
constructOptions(kButtonColors);
const page = document.getElementById("buttonDiv");
const kButtonColors = ["#3aa757", "#e8453c", "#f9bb2d", "#4688f1"];
function constructOptions(kButtonColors) {
  kButtonColors.forEach((color) => {
    const button = document.createElement("button");
    button.style.backgroundColor = color;
    button.addEventListener("click", function () {
      chrome.storage.sync.set({ color }, function () {
        console.log("color is " + color);
      });
    });
    page.appendChild(button);
  });
}
constructOptions(kButtonColors);
```

然后按照前面截图的步骤找到例子中扩展程序的选项页：

![](/madao.github.io/database/images/articles/google/getting_started_example/image11.png)

当你点击了其中任意一个和当前扩展程序中按钮颜色不同的按钮后，当前扩展程序的按钮就会变成你在选项页中点击的颜色，改变的页面背景色也会跟着改变。

### 三. 总结

1. 一个扩展是由不同的组件构成的，这些组件可以包含：

- background scripts （运行在后台的脚本）
- content scripts （运行在当前页的脚本）
- options page （选项页）
- UI elements （用户界面）
- 其他逻辑文件

2. 通过 background 可以定义一些常驻后台的脚本，这些脚本可分为持续运行和按需运行，推荐按需运行。

3. 可以通过配置 page_action 让扩展程序可以设置成在满足某些条件的页面上才能运行，并通过 chrome.declarativeContent 相关的 API 对规则进行设置。

4. 部分 Chrome 提供的 API 需要申请才能用，通过 permissions 字段进行申请。

5. 常驻后台的脚本可以通过 Chrome 提供的本地存储 API 和其他脚本进行通信。

6. Chrome 扩展程序中的 HTML 只能通过`<script src="xxx.js"></script>`这种方式引用 js 文件。

7. 扩展程序可以为用户提供一个选项页，让用户做一些个性化的配置，通过 options_page 字段进行配置

[源码](https://github.com/GreedyWhale/chrome_extensions_demo/tree/master/GettingStartedExample)
