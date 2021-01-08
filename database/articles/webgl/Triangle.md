### 一. 基本概念

由于完全没有学过图形学相关的知识，所以用 WebGL 时感到很吃力，对基本概念的理解也感觉不是很透彻，这里就不写我对 WebGL 的概念的理解了，推荐几篇文章：

[WebGL 教程
](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGL_API/Tutorial)

[GLSL 着色器](https://developer.mozilla.org/zh-CN/docs/Games/Techniques/3D_on_the_web/GLSL_Shaders)

[webgl 笔记-2.着色器和缓冲区](https://www.cnblogs.com/yiyezhai/archive/2012/09/21/2697461.html)

[GLSL 详解（基础篇）](https://colin1994.github.io/2017/11/11/OpenGLES-Lesson04/#%E6%A6%82%E8%A7%88)

就直接开始绘制吧，边绘制边说

### 二. 绘制三角形

- 第一步，获取 WebGL 的渲染上下文，之后的绘制都会在这个上下文中绘制，直白一点说就是拿到画布

  ```
  <body onload="main()">
      <canvas id="glcanvas" width="640" height="480"></canvas>
  </body>
  ```

  ```
  function main() {
      const canvas = document.querySelector('#glcanvas')
      const gl = canvas.getContext('webgl')
  }
  ```

- 第二步，创建着色器

  着色器就是绘制图形到屏幕上的函数，着色器运行在 GPU 中，编写着色器的语言是 GLSL，在 js 中需要将 GLSL 代码变成字符串，传递给 WebGL 的渲染上下文，使之在 GPU 执行时编译。

  着色器负责记录着像素点的位置和颜色。着色器分为两种：“顶点着色器”，“片元着色器（像素着色器）”

  - 顶点着色器：顶点着色器操作 3D 空间的坐标并且每个顶点都会调用一次这个函数. 其目的是设置 gl_Position 变量 -- 这是一个特殊的全局内置变量, 它是用来存储当前顶点的位置。例子：

    ![](/madao.github.io/database/images/articles/webgl/triangle/image.png)

    当绘制一个三角形的时候，会有上图这三个顶点，那么顶点着色器会在这三个顶点都执行一次，然后计算顶点的坐标并存起来。

  - 片元着色器，片元着色器会在绘制图形中的每个像素点调用一次。它的职责是确定像素的颜色。

  大概流程是这样的：

  1. 开发者确定绘制图形的顶点坐标，（我暂时理解为确定绘制图形的位置，大小）
  2. 通过 WebGL 的 api，将这些坐标传递给顶点着色器。
  3. 顶点着色器对接收到的坐标进行计算，转换成适用于屏幕上的坐标。并存在 gl_Position 中，gl_Position 是一个全局内置变量。
  4. 顶点着色器处理完成后，就会进行图元装配，WebGL 中可以直接绘制 7 种基本图形有：

     - gl.POINTS: 绘制一系列点。
     - gl.LINE_STRIP: 绘制一个线条。即，绘制一系列线段，上一点连接下一点。
     - gl.LINE_LOOP: 绘制一个线圈。即，绘制一系列线段，上一点连接下一点，并且最后一点与第一个点相连。
     - gl.LINES: 绘制一系列单独线段。每两个点作为端点，线段之间不连接。
     - gl.TRIANGLE_STRIP：绘制一个三角带。
     - gl.TRIANGLE_FAN：绘制一个三角扇。
     - gl.TRIANGLES: 绘制一系列三角形。每三个点作为顶点。

     ![](/madao.github.io/database/images/articles/webgl/triangle/image1.png)

  5. 光栅化，光栅化就是根据顶点信息加上图元将绘制一个图形所需要的像素点全部计算出来，交给片元着色器处理。

     就像这样：

     ![](/madao.github.io/database/images/articles/webgl/triangle/image2.png)

  6. 片元着色器会对每一个光栅化得到的每一个像素点进行着色。

     ![](/madao.github.io/database/images/articles/webgl/triangle/image3.png)

  ```

  // 创建着色器

  function createShader(gl, sourceCode, type) {

      // 创建shader

      const shader = gl.createShader(type)

      // 传入shader的源代码，源代码使用GLSL编写，通过字符串的形式传递进来

      gl.shaderSource(shader, sourceCode)

      // 编译shader代码

      gl.compileShader(shader)

      return shader

  }

  // 顶点着色器的源码，暂时理解为固定的格式，必须要有main函数

  const VSHADER_SCURCE_CODE = `

      // attribute 和 vec4 得去看GLSL的文档，我目前无法很好的解释

      // a_Position 通过js传递

      attribute vec4 a_Position;

      void main() {

          gl_Position = a_Position;

      }

  `

  // 例子中的每个像素值都是红色，所以这里直接写死颜色，颜色的格式是rgba

  const FSHADER_SCURCE_CODE = `

      void main() {

          gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);

      }

  `

  // 定义着色器

  const vertexShader = createShader(gl, VSHADER_SCURCE_CODE, gl.VERTEX_SHADER)

  const fragmentShader = createShader(gl, FSHADER_SCURCE_CODE, gl.FRAGMENT_SHADER)

  ```

- 第三步，将一个顶点着色器和一个片元着色器组成一个着色器程序，绑定到 WebGL 的渲染上下文中：

  ```

  // 创建一个着色器程序

  const program = gl.createProgram()

  // 给着色器程序添加着色器

  gl.attachShader(program, vertexShader)

  gl.attachShader(program, fragmentShader)

  // 将着色器程序绑定到webgl

  gl.linkProgram(program)

  gl.useProgram(program)

  gl.program = program

  ```

- 第四步，向顶点着色器中传递顶点坐标

  ```

  function initVertexBuffers(gl) {

      // 定义顶点坐标，格式是Float32Array 类型数组

      const vertices = new Float32Array([

          -1, 1,

          -1, -1,

           1, -1

      ])

      // 创建缓冲区，缓冲区我的理解是js将数据存在里面，着色器才可以拿的到数据

      const vertexBuffer = gl.createBuffer()

      // 绑定缓存区到当前webgl渲染上下文

      gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer)

      // 将坐标值传入缓冲区

      gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW)

      // 获取a_Position在缓冲区的地址，目的是确定传入的坐标应用到顶点着色器中的哪一个变量

      const a_position = gl.getAttribLocation(gl.program, 'a_Position')

      // 给a_position传递坐标值

      // vertexAttribPointer第一个参数就是接收坐标值的变量位置。

      // 第二个参数告诉每次取几个坐标

      // 第三个参数是用于确定数据的格式

      // 第四个参数 指定当被访问时，固定点数据值是否应该被归一化（GL_TRUE）或者直接转换为固定点值（GL_FALSE）；

      // 第五个参数说明数据存储的方式，单位是字节。0表示一个属性的数据是连续存放的，非0则表示同一个属性在数据中的间隔大小。也就是一个顶点的数据占用了多少字节。

      // 第六个参数属性在缓冲区的偏移值，单位是字节。0表示从头开始读取。

      // 第四，五，六个参数，我暂时还不能理解是干嘛的。。。

      gl.vertexAttribPointer(a_position, 2, gl.FLOAT, false, 0, 0)

      // 让顶点着色器的a_position可以从缓冲区中拿数据

      gl.enableVertexAttribArray(a_position)

      // 这里返回的3是为了指定有多少个顶点

      return 3

  }

  ```

- 第五步，进行绘制

  ```

  // 获取顶点的数量

  const n = initVertexBuffers(gl)

  // 指定设置清空颜色缓冲区时的颜色值。

  gl.clearColor(0.0, 0.0, 0.0, 1.0)

  // 绘制

  function draw() {

      // 清空上一次的绘制，因为是逐帧绘制，所以每一次绘制都要清除上次绘制的内容。

      gl.clear(gl.COLOR_BUFFER_BIT)

      // 绘制传入的图元

      // 第一个参数就是图元的类型，上面有说到

      // 第二个参数是指定从哪个顶点开始

      // 第三个参数则是顶点的数量

      gl.drawArrays(gl.TRIANGLES, 0, n)

  }

  draw()

  ```

- 效果：

  ![](/madao.github.io/database/images/articles/webgl/triangle/image4.png)

- 完整代码：

  html

  ```

  <!DOCTYPE html>

  <html lang="en">

  <head>

      <meta charset="UTF-8">

      <meta name="viewport" content="width=device-width, initial-scale=1.0">

      <meta http-equiv="X-UA-Compatible" content="ie=edge">

      <title>Document</title>

  </head>

  <body onload="main()">

      <canvas id="glcanvas" width="640" height="480"></canvas>

      <script type="text/javascript" src="./index.js"></script>

  </body>

  </html>

  ```

  js

  ```

  function main() {

      const canvas = document.querySelector('#glcanvas')

      const gl = canvas.getContext('webgl')

      const program = gl.createProgram()

      function createShader(gl, sourceCode, type) {

          const shader = gl.createShader(type)

          gl.shaderSource(shader, sourceCode)

          gl.compileShader(shader)

          return shader

      }

      const VSHADER_SCURCE_CODE = `

          attribute vec4 a_Position;

          void main() {

              gl_Position = a_Position;

          }

      `

      const FSHADER_SCURCE_CODE = `

          void main() {

              gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0);

          }

      `

      const vertexShader = createShader(gl, VSHADER_SCURCE_CODE, gl.VERTEX_SHADER)

      const fragmentShader = createShader(gl, FSHADER_SCURCE_CODE, gl.FRAGMENT_SHADER)

      gl.attachShader(program, vertexShader)

      gl.attachShader(program, fragmentShader)

      gl.linkProgram(program)

      gl.useProgram(program)

      gl.program = program

      function initVertexBuffers(gl) {

          const vertices = new Float32Array([

              -1, 1,

              -1, -1,

              1, -1

          ])

          const vertexBuffer = gl.createBuffer()

          gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer)

          gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW)

          const a_position = gl.getAttribLocation(gl.program, 'a_Position')

          gl.vertexAttribPointer(a_position, 2, gl.FLOAT, false, 0, 0)

          gl.enableVertexAttribArray(a_position)

          return 3

      }

      const n = initVertexBuffers(gl)

      gl.clearColor(0.0, 0.0, 0.0, 1.0)

      function draw() {

          gl.clear(gl.COLOR_BUFFER_BIT)

          gl.drawArrays(gl.TRIANGLES, 0, n)

      }

      draw()

  }

  ```

- WebGL 中的坐标系

  WebGL 中的坐标系是 3d 的，也就是有 x,y,z 三个坐标轴：

  看这篇文章就明白了[WebGL 的坐标系统](https://blog.csdn.net/qq_30100043/article/details/70878160)

  然后每个轴最大值是 1，最小值是-1，坐标原点在 canvas 元素的中心点。
