先上最终效果

![](/madao.github.io/database/images/articles/gadgets/canvas_map/map.gif)

[源码](https://github.com/GreedyWhale/canvas_map_effects)

就是这样一个从多个点连接到一个点的动画效果。

我本以为 canvas 的动画是和 css 的一样，写好动画关键帧、时间、重复次数这些，然后就可以动起来了，但实际上不是这样的，canvas 的动画效果需要写出动画的每一帧，然后不断的重新渲染画布从而实现动画。

上图的圆形扩散就是从半径为 0 开始画圆，然后每次增加圆的半径，再降低透明度实现，所以 canvas 实现动画效果是要不断的重新渲染画面

### 实现

#### 1. 添加背景

例子中的背景是一张图片，所以直接使用 drawImage 在 canvas 上绘制即可，但是这里需要注意的是，canvas 元素的 with，height 属性，和 css 中的 with，height 属性是不一样的。

canvas 元素的 with，height 属性`<canvas width=xxx height=xxx>`规定的 canvas 画布的大小，而使用 css 的 with，height 属性规定的是呈现给用户的大小，当这两个尺寸不一致时，canvas 元素会根据 css 的 with，height 进行缩放，绘制出的图形就会变形，当然如果是整数倍那么是没问题的，比如 canvas 的 width 是 400，height 是 200，css 的 width 是 200，height 是 100，这样是没问题的，像这种 css 的 width，height 小于 canvas 的 width、height 的设置，还有助于提高绘制图片的清晰度。

参考: [canvas HTML 属性尺寸和 CSS 尺寸多个细节深入](https://www.zhangxinxu.com/wordpress/2018/07/canvas-html-size-css-size/)

- ##### 新建一个 index.html
  ```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="UTF-8" />
      <meta name="viewport" content="width=device-width, initial-scale=1.0" />
      <title>Demo</title>
    </head>
    <body>
      <canvas class="back-canvas"></canvas>
      <script src="./main.js"></script>
    </body>
  </html>
  ```
- ##### 新建一个 main.js

  ```javascript
  (function () {
    const app = {
      backCanvas: document.querySelector(".back-canvas"),
      backContext: null,
      ratio: 2,

      // 做数值转换的原因是，绘制出的图形太模糊了，所以使用2倍图
      numericalConversion(number) {
        return number * this.ratio;
      },

      drawBg() {
        return new Promise((resolve) => {
          const bgImage = new Image();
          // 背景图片
          bgImage.src = "./assets/images/map.png";
          bgImage.onload = () => {
            const { naturalWidth, naturalHeight } = bgImage;
            const width = this.numericalConversion(naturalWidth);
            const height = this.numericalConversion(naturalHeight);
            // 设置canvas的HTML属性width，height
            this.backCanvas.width = width;
            this.backCanvas.height = height;
            // 设置canvas的css属性width，height
            this.backCanvas.style.width = `${naturalWidth}px`;
            this.backCanvas.style.height = `${naturalHeight}px`;

            this.backContext.drawImage(bgImage, 0, 0, width, height);
            resolve();
          };
        });
      },

      init() {
        this.backContext = this.backCanvas.getContext("2d");
        // 画背景
        this.drawBg();
      },
    };

    app.init();
  })();
  ```

  注意上面的代码中我把 canvas 元素的 HTML 属性 width，height 设置成了它 css 属性 width，height 的两倍，这是因为我在第一次实现的时候，发现画出来的图形太模糊了，正好张鑫旭大神的文章中有说这种情况的解决办法，就按照他说的方法试了下，果然是可以的。

  目前的效果是这样的（背景图片可自行修改），就是一张地图

  ![](/madao.github.io/database/images/articles/gadgets/canvas_map/map.png)

#### 2. 实现地图上闪烁的点

- ##### index.html

  ```diff
  <!--之前代码保不变-->
  <canvas class="back-canvas"></canvas>
  + <canvas class="point-canvas"></canvas>
  ```

  在之前的 back-canvas 元素后面再添加一个 point-canvas 元素，用于绘制闪烁的点

- ##### 新建 style.css

  因为我是分多个 canvas 进行绘制，所以要让它们的位置重叠才行，不然点和背景就对不上了。

  ```css
  body {
    position: relative;
  }
  canvas {
    position: absolute;
    top: 0;
    left: 0;
  }
  canvas:nth-of-type(1) {
    z-index: 1;
  }
  canvas:nth-of-type(2) {
    z-index: 2;
  }
  ```

  记得在 index.html 中引入 style.css

- ##### main.js

  1. 先确定每个点的坐标

     ```javascript
     // 之前代码保持不变
     const app = {
       coordinateGroup: [
         { x: 1198, y: 640 },
         { x: 1129, y: 270 },
         { x: 1128, y: 394 },
         { x: 1025, y: 383 },
         { x: 841, y: 317 },
         { x: 756, y: 276 },
         { x: 700, y: 500 },
         { x: 480, y: 200 },
         { x: 430, y: 400 },
         { x: 470, y: 600 },
         { x: 250, y: 360 },
       ],

       setCoordinateGroup() {
         this.coordinateGroup = this.coordinateGroup.map(({ x, y }) => ({
           x: this.numericalConversion(x),
           y: this.numericalConversion(y),
         }));
       },

       init() {
         this.setCoordinateGroup();
       },
     };
     ```

  2. 初始化圆点的画布

     ```javascript
     const app = {
       pointCanvas: document.querySelector(".point-canvas"),
       pointContext: null,

       drawBg() {
         return new Promise((resolve) => {
           const bgImage = new Image();
           bgImage.src = "./assets/images/map.png";
           bgImage.onload = () => {
             const { naturalWidth, naturalHeight } = bgImage;
             const width = this.numericalConversion(naturalWidth);
             const height = this.numericalConversion(naturalHeight);
             [this.backCanvas, this.pointCanvas].forEach((value) => {
               // 设置canvas的HTML属性width，height
               value.width = width;
               value.height = height;
               // 设置canvas的css属性width，height
               value.style.width = `${naturalWidth}px`;
               value.style.height = `${naturalHeight}px`;
             });

             this.backContext.drawImage(bgImage, 0, 0, width, height);
             resolve();
           };
         });
       },

       init() {
         this.pointContext = this.pointCanvas.getContext("2d");
       },
     };
     ```

  3. 实现画圆的方法

     ```javascript
     const app = {
       drawCircle() {
         this.coordinateGroup.forEach(({ x, y }) => {
           this.pointContext.beginPath();
           this.pointContext.arc(x, y, 50, 0, 2 * Math.PI);
           this.pointContext.closePath();
           this.pointContext.strokeStyle = "#005086";
           this.pointContext.lineWidth = 4;
           this.pointContext.stroke();
         });
       },
     };
     ```

  4. 将圆画出来

     ```javascript
     const app = {
       init() {
         this.pointContext = this.pointCanvas.getContext("2d");
         // 画背景
         this.drawBg().then(() => {
           // 画圆
           this.drawCircle();
         });
       },
     };
     ```

     目前的效果：

     ![](/madao.github.io/database/images/articles/gadgets/canvas_map/map2.png)

     上面说过这种闪烁的其实是不断的变化圆的半径和透明度实现的，所以这里还需要一个可以变化的半径

  5. 实现圆点半径的动态改变

     ```javascript
     // 修改一下drawCircle方法

     drawCircle () {
       this.coordinateGroup = this.coordinateGroup.map(({ x, y, radius = 0 }) => {
         this.pointContext.beginPath()
         this.pointContext.arc(x, y, radius, 0, 2 * Math.PI)
         this.pointContext.closePath()
         this.pointContext.strokeStyle = '#ed6663'
         this.pointContext.lineWidth= 4
         this.pointContext.stroke()
         radius += 1
         if (radius > 60) {
           radius = 0
         }
         return { x, y, radius }
       })
     },
     ```

     上面在每次画圆的同时，更新坐标，给半径一个最大值和每次增加加的量，当超过最大值后归零。

  6. 实现闪烁效果

     ```javascript
     const app = {
         drawBg () {
             return new Promise(resolve => {
               const bgImage = new Image()
               bgImage.src = './assets/images/map.png'
               bgImage.onload = () => {
                 const { naturalWidth, naturalHeight } = bgImage
                 const width = this.numericalConversion(naturalWidth)
                 const height = this.numericalConversion(naturalHeight)
                 ;[this.backCanvas, this.pointCanvas].forEach(value => {
                   // 设置canvas的HTML属性width，height
                   value.width = width
                   value.height = height
                   // 设置canvas的css属性width，height
                   value.style.width = `${naturalWidth}px`
                   value.style.height = `${naturalHeight}px`
                 })

                 this.backContext.drawImage(bgImage, 0, 0, width, height)
                 this.pointContext.globalAlpha = 0.95 // 重要，重要，重要！！！
                 resolve()
               }
             })
         },
         renderPoint () {
           this.pointContext.globalCompositeOperation = 'destination-in'
           this.pointContext.fillRect(0, 0, this.pointCanvas.width, this.pointCanvas.height)
           this.pointContext.globalCompositeOperation = 'source-over'
           this.drawCircle()
         }，
         render () {
           this.renderPoint()
           window.requestAnimationFrame(this.render.bind(this))
         },
         init () {
         	init () {
             this.backContext = this.backCanvas.getContext('2d')
             this.pointContext = this.pointCanvas.getContext('2d')
             this.setCoordinateGroup()
             // 画背景
             this.drawBg().then(() => {
               // 启动动画
               this.render()
             })
           }
         }
     }
     ```

     效果：

     ![](/madao.github.io/database/images/articles/gadgets/canvas_map/map1.gif)

- ##### 解释下代码：

  1.  CanvasRenderingContext2D.globalCompositeOperation = 'destination-in'

      globalCompositeOperation 是用来设置在 canvas 上绘制新图形时，和已有的图形要怎么混合的属性，destination-in 指的是只保留已有图形和新图形重叠的部分，例子：

      默认情况下在画布上绘制重叠的图形的时候会是这样的效果：

      ![](/madao.github.io/database/images/articles/gadgets/canvas_map/example.png)

      当 globalCompositeOperation 改为'destination-in'时：

      ![](/madao.github.io/database/images/articles/gadgets/canvas_map/example1.png)

      这里就剩下蓝色块和红色块重叠的部分了，并且显示是已有的图形，而不是新绘制的图形。

      ```
      this.pointContext.globalCompositeOperation = 'destination-in'
      this.pointContext.fillRect(0, 0, this.pointCanvas.width, this.pointCanvas.height)
      ```

      这两句的代码的意思就是把之前画出的圆保存起来。

  2.  this.pointContext.globalAlpha = 0.95

      这一句是非常重要的一句代码，也是我理解最久的一句，globalAlpha 是用来设置整个画布的透明度的，只要你设置了这个属性，之后绘制的图形都会有设置的透明度

      这里也是让我最不能理解的一点，当相同的透明度不断的进行叠加的时候，理论上是越来越不透明才对，而且 MDN 上的例子也是这样：

      ![](/madao.github.io/database/images/articles/gadgets/canvas_map/example2.png)

      后来咨询了一下公司的大神，我是这样向我解释的，当你用 80%的透明度画不同颜色的两个方块，然后把它合成一张图片，再给这张图片加上一个 80%的透明度，那么这两个方块的透明度就成了 80% \* 80% = 64%，将这个理论对应到 canvas 中就是：`this.pointContext.globalAlpha = 0.95`，设置画布的全局透明度为 0.95，画出第一个圆，然后再将这个圆保存起来

      ```
      this.pointContext.globalCompositeOperation = 'destination-in'
      this.pointContext.fillRect(0, 0, this.pointCanvas.width, this.pointCanvas.height)
      ```

      在切换 globalCompositeOperation 为默认值`source-over`，canvas 会进行一次合并，就是 0.95 \* 0.95，然后再在合并后的画布上继续绘制，这样原来的圆的透明度就下降了，最后会越来越小直到完全透明。

      这个只是个人理解，不知道对不对。

  3.  window.requestAnimationFrame(this.render.bind(this))

      上面也说过，动画是通过不断渲染实现的，这个就是用来进行重复渲染的方式，当然也可以改成 setTimeout 或 setInterval 来实现。

#### 3. 绘制点与点之间的连线

点与点之间使用曲线进行连接，canvas 提供了一个方法`quadraticCurveTo`，用于绘制二次贝赛尔曲线，我也不明白二次贝赛尔曲线是什么，反正就是提供起点，终点和控制点，它就可以根据这三个点绘制出一段平滑的曲线，在这里面起点就是各个坐标的值，终点可以随便选一个坐标，这里我选择地图中接近北京的坐标点，控制点每条线都不一样，所以需要绘制的时候确定，x 我用的是点与点之间的一半，y 是起点坐标减去 100。

- #### main.js

  ```javascript
  const app = {
    centerCoordinate: null,

    isCenterPoint(x, y) {
      return x === this.centerCoordinate.x && y === this.centerCoordinate.y;
    },

    drawLine() {
      this.coordinateGroup.forEach(({ x, y }) => {
        // 中心点与中心点之间不需要连线
        if (this.isCenterPoint(x, y)) {
          return;
        }
        this.backContext.strokeStyle = "#b52b65";
        this.backContext.lineWidth = this.numericalConversion(1);
        this.backContext.moveTo(x, y);
        const controlPoint = {
          x: (x + this.centerCoordinate.x) / 2,
          y: y - this.numericalConversion(100),
        };
        this.backContext.quadraticCurveTo(
          controlPoint.x,
          controlPoint.y,
          this.centerCoordinate.x,
          this.centerCoordinate.y
        );
        this.backContext.stroke();
      });
    },

    init() {
      this.backContext = this.backCanvas.getContext("2d");
      this.pointContext = this.pointCanvas.getContext("2d");
      this.setCoordinateGroup();
      // 设置中心点坐标
      this.centerCoordinate = {
        x: this.numericalConversion(1025),
        y: this.numericalConversion(383),
      };
      // 画背景
      this.drawBg().then(() => {
        // 绘制连线
        this.drawLine();
        // 启动动画
        this.render();
      });
    },
  };
  ```

  效果：

  ![](/madao.github.io/database/images/articles/gadgets/canvas_map/map2.gif)

#### 4. 实现在曲线上移动的小点

由于曲线是用二次贝赛尔曲线实现的，它是有公式的，可以根据不同时间点的坐标，从网上抄一个

```
/**
  * 二次贝塞尔曲线方程
  * @param  {number} startPoint 起点
  * @param  {number} controlPoint 曲度点
  * @param  {number} endPoint 终点
  * @param  {number} progress 当前百分比(0-1)
  * @returns {number}
  */
  quadraticBezier (startPoint, controlPoint, endPoint, progress) {
    const { x: startPointX, y: startPointY } = startPoint;
    const { x: endPointX, y: endPointY } = endPoint;
    const { x: controlPointX, y: controlPointY } = controlPoint;
    const t = 1 - progress
    const x = t * t * startPointX + 2 * progress * t * controlPointX + progress * progress * endPointX
    const y = t * t * startPointY + 2 * progress * t * controlPointY + progress * progress * endPointY
    return { x, y }
  }
```

- ##### index.html

  小圆点是运动的所以我又使用了一个 canvas 元素来绘制它

  ```
  <canvas class="back-canvas"></canvas>
  <canvas class="point-canvas"></canvas>
  <canvas class="track-canvas"></canvas>  // 新增
  ```

- ##### style.css

  ```
  canvas:nth-of-type(3) {
    z-index: 2;
  }
  ```

- ##### main.js

  ```javascript
  const app = {
    trackCanvas: document.querySelector(".track-canvas"),
    trackContext: null,
    progress: 0, // 小圆点运动的进度

    // 由于连线和运动轨迹的绘制都需要这个控制点，所以封装一下
    getControlPoint(x, y) {
      const controlPoint = {
        x: (x + this.centerCoordinate.x) / 2,
        y: y - this.numericalConversion(100),
      };
      return controlPoint;
    },

    drawBg() {
      return new Promise((resolve) => {
        const bgImage = new Image();
        bgImage.src = "./assets/images/map.png";
        bgImage.onload = () => {
          const { naturalWidth, naturalHeight } = bgImage;
          const width = this.numericalConversion(naturalWidth);
          const height = this.numericalConversion(naturalHeight);
          // 增加trackCanvas的width、height属性设置
          [this.backCanvas, this.pointCanvas, this.trackCanvas].forEach(
            (value) => {
              // 设置canvas的HTML属性width，height
              value.width = width;
              value.height = height;
              // 设置canvas的css属性width，height
              value.style.width = `${naturalWidth}px`;
              value.style.height = `${naturalHeight}px`;
            }
          );

          this.backContext.drawImage(bgImage, 0, 0, width, height);
          this.pointContext.globalAlpha = 0.95;
          resolve();
        };
      });
    },

    drawLine() {
      this.coordinateGroup.forEach(({ x, y }) => {
        // 中心点与中心点之间不需要连线
        if (this.isCenterPoint(x, y)) {
          return;
        }
        this.backContext.strokeStyle = "#065446";
        this.backContext.lineWidth = this.numericalConversion(1);
        this.backContext.moveTo(x, y);
        const controlPoint = this.getControlPoint(x, y); // 这里替换成封装好的方法
        this.backContext.quadraticCurveTo(
          controlPoint.x,
          controlPoint.y,
          this.centerCoordinate.x,
          this.centerCoordinate.y
        );
        this.backContext.stroke();
      });
    },
    // 画运动轨迹
    drawTrack() {
      this.trackContext.clearRect(
        0,
        0,
        this.backCanvas.width,
        this.backCanvas.height
      );
      this.coordinateGroup.forEach(({ x, y }) => {
        if (this.isCenterPoint(x, y)) {
          return;
        }
        const controlPoint = this.getControlPoint(x, y);
        const endPoint = this.quadraticBezier(
          { x, y },
          controlPoint,
          this.centerCoordinate,
          this.progress / 100
        );
        const color = "#ed6663";
        this.trackContext.beginPath();
        this.trackContext.shadowColor = color;
        this.trackContext.shadowBlur = this.numericalConversion(8);
        this.trackContext.fillStyle = color;
        this.trackContext.arc(
          endPoint.x,
          endPoint.y,
          this.numericalConversion(4),
          0,
          2 * Math.PI
        );
        this.trackContext.fill();
      });
    },

    rendTrack() {
      if (this.progress <= 100) {
        this.drawTrack();
        this.progress += 1;
        return;
      }
      setTimeout(() => {
        this.progress = 0;
        this.drawTrack();
      }, 200);
    },

    render() {
      this.renderPoint();
      this.rendTrack(); // 新增运动轨迹的绘制
      window.requestAnimationFrame(this.render.bind(this));
    },
  };
  ```

  到这里所有的代码就都写完了，其实难点就是要理解 canvas 的 globalCompositeOperation 属性，虽然不知道我现在理解的是否正确，但是从逻辑上解释的通，然后曲线的绘制就简单了，主要是二次贝塞尔曲线。

  最后所有的代码都在这里：[源码](https://github.com/GreedyWhale/canvas_map_effects)
