### Hello Extensions

最近需要开发一个 Chrome 插件，看了一些教程，还是决定跟着官方教程学习。

### 一. 项目搭建

1. 新建 HelloExtensions 文件夹，作为项目目录
2. 在项目目录中创建 manifest.json 文件
   ```
   {
       "name": "Hello Extensions",
       "description" : "Base Level Extension",
       "version": "1.0",
       "manifest_version": 2,
       "browser_action": {
           "default_popup": "hello.html",
           "default_icon": {
               "16": "images/logo16.png",
               "24": "images/logo24.png",
               "32": "images/logo32.png"
           }
       },
       "commands": {
           "_execute_browser_action": {
               "suggested_key": {
                   "default": "Ctrl+Shift+1",
                   "mac": "MacCtrl+Shift+1"
               },
               "description": "Opens hello.html"
           }
       }
   }
   ```
3. 项目目录中创建 hello.html
   ```
   <html>
       <body>
           <h1>Hello Extensions</h1>
       </body>
   </html>
   ```
4. 现在项目目录看起来是这样的：

![](/madao.github.io/database/images/articles/google/hello_extensions/image.png)

### 二. 查看效果

1. 在 Chrome 的地址栏里输入：chrome://extensions
2. 在页面的左上角开启开发者模式：
   ![](/madao.github.io/database/images/articles/google/hello_extensions/image1.png)
3. 开启开发者模式后，页面上就会出现`加载已解压的扩展程序`这个按钮，点击这个按钮选择之前的项目文件夹，就会出现：
   ![](/madao.github.io/database/images/articles/google/hello_extensions/image2.png)
4. 在 Chrome 的地址栏左侧可以看到这个扩展程序的图标，点击这个图标，或者用快捷键`Ctrl+Shift+1`，就可以看到 hello.html 这个页面：
   ![](/madao.github.io/database/images/articles/google/hello_extensions/image3.png)

### 三. 总结

这个项目来源于官方文档，非常简单，总结下学习到的知识点：

1. Chrome 扩展程序使用基本的 Web 技术进行开发：HTML、JavaScript、CSS
2. 每个扩展程序都必须有 manifest.json 这个文件，manifest.json 就像是项目的配置文件
3. manifest.json 必须包含的字段：
   - manifest_version：从 Chrome 18 开始这个值必须为 2，所以一般填 2 就可以了
   - name：扩展程序的名字
   - version：扩展程序的版本号，不能以 0 开头，以下都是合法值：
     ```
     "version": "1"
     "version": "1.0"
     "version": "2.10.2"
     "version": "3.1.2.4567"
     ```
     每个区间取值范围是：0 到 65535。
4. manifest.json 中除了必填的那三个字段之外，这个项目还涉及到了以下字段：
   - description：扩展程序的描述
   - browser_action：Chrome 的扩展程序有所有页面都可以用的，也有指定页面才可以用的，比如 Vue、React 的调试工具，它在特定的页面才可以用，browser_action 这个字段则是定义扩展程序在所有的页面上都可以用，与之对应的是 page_action，它规定了扩展程序在满足特定条件的页面才可用，browser_action 包含三个属性：
     - default_icon：用于定义显示在 Chrome 工具栏（地址栏左侧）的图标路径，可以传入多个尺寸的图标，Chrome 会选择最合适的用，图标的尺寸最好是能填充满 16 \* 16 dip (dip 是 设备独立像素 的意思)
     - default_popup：鼠标点击工具栏上扩展程序图标要展示的页面路径
     - default_title：鼠标悬浮在工具栏上扩展程序图标要展示的文字
   - commands：用于定义一些快捷键的字段，这个项目中有用到\_execute_browser_action 这个属性，这个属性中定义的快捷键的效果就和用鼠标点击工具栏上扩展程序图标一样，我在测试的时候发现，如果定义的快捷键和现有的有冲突，那么它是不会生效的，除了\_execute_browser_action 之外还有\_execute_page_action 这个属性，虽然这个项目没有用到，但是大概也能猜到，它是用于定义 page_action 的这个字段的快捷键，还可以自己定义一些其他的快捷键，目前我还没有用过，暂时就先不写了。

[源码](https://github.com/GreedyWhale/chrome_extensions_demo/tree/master/HelloExtensions)

### 学习资料

[What are extensions?](https://developer.chrome.com/extensions)

[Chrome 扩展及应用开发（首发版）](https://www.ituring.com.cn/book/miniarticle/110853)

[一篇文章教你顺利入门和开发 chrome 扩展程序（插件）](https://juejin.im/post/6844903740646899720#heading-51)

[manifest.json 所有配置官方文档](https://developer.chrome.com/extensions/manifest)
