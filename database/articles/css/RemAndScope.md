### 一. 动态 rem 的计算公式

设计稿宽度/页面根元素宽度 * 基准字体大小
*注意：基准字体大小不要设置为 12px 以下，因为 chrome 浏览器最小字体大小为 12px，最好设置成好换算的数值\*

```

// 100px = 1rem

(function() {

  // 获取html元素

  const rootNode = document.documentElement;

  // 判断页面改变大小的事件类型

  const eventType = "onorientationchange" in window ? "orientationchange" : "resize";

  // 设计稿宽度

   const designWidth = 750;

   // 基准字体大小

   const baseFontsize = 100;

   function setFontsize() {

     const clientWidth = window.screen.availWidth;

     rootNode.style.fontSize = (clientWidth / designWidth) * baseFontsize + 'px';

   }

   window.addEventListener(eventType, setFontsize, false);

   document.addEventListener("DOMContentLoaded", setFontsize, false);

})()

```

### 二. css 变量

原来 css 是支持变量的，我居然今天才知道。。
[CSS 变量教程](http://www.ruanyifeng.com/blog/2017/05/css-variables.html)

一些公共的样式就可放在:root 选择器中了（:root 选择器也是今天才知道的。相当于 html 标签选择器），用到的地方用 var 函数获取。

```

:root {

  --fontsize: 16px;

  --color: red;

}

.box {

  font-size: var(--fontsize);

  color: var(--color);

}

```
