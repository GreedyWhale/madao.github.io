three.js 是一个可以使用 javascript 绘制 3d 图形的库，它对 WebGL 的 api 进行封装，使开发更加方便，就像 jQuery 对 DOM 的 api 进行封装一样。接下来就记录一下在小游戏中绘制一个
旋转的三角形的步骤：

### 一. 初始化项目

下载微信官方的[开发者工具](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html)，然后新建项目

![](/madao.github.io/database/images/articles/threejs/triangle/image.png)

appid 选择测试号即可，项目路径自行指定

用编辑器打开项目，得到如下目录：

![](/madao.github.io/database/images/articles/threejs/triangle/image1.png)

然后除了 game.js，game.json, project.config.json 全部删除，并把 game.js 中的内容清空。

![](/madao.github.io/database/images/articles/threejs/triangle/image2.png)

game.js 是整个小游戏的入口，game.json 是小游戏配置。具体参考[文档](https://developers.weixin.qq.com/minigame/dev/guide/framework/config.html)。

### 二. 引入 three.js 和 Adapter

- Adapter

  小游戏的运行环境中是没有 BOM 和 DOM 的，使用 wx API 模拟 BOM 和 DOM 的代码组成的库称之为 Adapter。官方提供了一个 Adapter，用它就行了。

  [Adapter 文档](https://developers.weixin.qq.com/minigame/dev/guide/best-practice/adapter.html)

- three.js

  [gitHub 地址](https://github.com/mrdoob/three.js)

  ![](/madao.github.io/database/images/articles/threejs/triangle/image3.png)

  复制 three.min.js 中的内容

新建目录 libs，将 three.js 和 Adapter 的源码放在该目录下

![](/madao.github.io/database/images/articles/threejs/triangle/image4.png)

在 game.js 中添加：

```
import './libs/weapp-adapter'
import * as THREE from './libs/three'
```

### 三. 绘制三角形

根据 adapter 的文档只要引入了 adapter 就会创建一个上屏 Canvas，并暴露为一个全局变量 canvas。

使用 three.js 渲染一个图形必备的三个条件：渲染器，场景，相机

- Renderer 渲染器
  渲染器看名字就知道了，就是用于将图形渲染到屏幕上的方法。
- Scene 场景
  假如把绘制的图形看做是一个个物体的话，那么场景就是用来存放这些物体的地方。
- Camera 相机
  相机就好像人的眼睛一样，相机用于确定在什么地方去看场景中的物体，就好像有一个东西，不同的角度去看这个物体，看到的有可能是不一样的形状。

在 game.js 中创建这三个东西

```
import './libs/weapp-adapter'
import * as THREE from './libs/three'

const width = window.innerWidth
const height = window.innerHeight
// 创建WebGL渲染器
const renderer = new THREE.WebGLRenderer({
  // 由于weapp-adapter会自动创建一个全屏的canvas所以这里直接用
  canvas
})

// 创建场景
const scene = new THREE.Scene()

/**
 * OrthographicCamera是正交相机，
 * 在这种投影模式下，无论物体距离相机距离远或者近，在最终渲染的图片中物体的大小都保持不变。
*/
const camera = new THREE.OrthographicCamera(-width / 2, width / 2, height / 2, -height / 2, 0, 1000)
```

`new THREE.OrthographicCamera` 的参数可以参考[官方文档](https://threejs.org/docs/index.html#api/zh/cameras/OrthographicCamera)或者[Three.js 基础探寻二——正交投影照相机](https://www.cnblogs.com/xulei1992/p/5707733.html)

现在必要的三个条件都有了，就要添加物体到场景中了。

物体在 three.js 中叫做 mesh，它由几何体（geometry）和材料（material）组成。

我的理解就是几何体就是物体的基本形状，就像 WebGL 中的顶点着色器，材料就是几何体的颜色啊，光照等信息，就像 WebGL 中的片元着色器。

three.js 中提供了很多几何体，但是好像没有基本的三角形，所以要自己画一个三角形。

在 game.js 中添加：

```
// 画一个三角形
const triangleShape = new THREE.Shape()
triangleShape.moveTo(0, 100)  // 三角形起始位置
triangleShape.lineTo(-100, -100)
triangleShape.lineTo(100, -100)
triangleShape.lineTo(0, 100)
```

这里说一下 three.js 的坐标系

![](/madao.github.io/database/images/articles/threejs/triangle/image5.png)

有了三角形的基本形状，通过 three.js 中提供的 api，将这个三角形变成几何体

在 game.js 中添加：

```
// 将三角形变成组成物体的几何体
const geometry = new THREE.ShapeGeometry(triangleShape)
```

组成物体的几何体就搞定了。

然后就是材料了：

在 game.js 中添加：

```
// 物体的材料
const material = new THREE.MeshBasicMaterial({
  color: new THREE.Color('#7fffd4'),  // 颜色信息
  side: THREE.DoubleSide         // 用于确定渲染哪一面，因为是旋转的，所以需要正反面都渲染，也就是两面
})
```

用几何体 + 材料组成物体，并添加到场景中：

```
// 组成物体并添加到场景中
const mesh = new THREE.Mesh(geometry, material)
mesh.position.set(0, 0, -200)  // 设置物体在场景中的位置
scene.add(mesh)
```

设置相机的位置以及看向的坐标

```
camera.position.set(0, 0, 0)  // 相机位置
camera.lookAt(new THREE.Vector3(0, 0, -200)) // 让相机从0, 0, 0 看向 0, 0, -200
```

最后一步就是渲染了：

```
renderer.setClearColor(new THREE.Color('#f84462')) // 设置背景色
renderer.setSize(width, height) // 设置最终渲染的尺寸
renderer.render(scene, camera)
```

这时候去在开发者工具中就可以看到一个三角形了：

![](/madao.github.io/database/images/articles/threejs/triangle/image6.png)

### 四. 让三角形动起来

```
const render = () => {
  mesh.rotateY(0.05)  // three.js 中旋转角度是通过弧度计算的，公式：度＝弧度×180°/π
  renderer.render(scene, camera)
  requestAnimationFrame(render)
}

render()
```

完整代码：

```

import './libs/weapp-adapter'

import * as THREE from './libs/three'

const width = window.innerWidth

const height = window.innerHeight

// 创建WebGL渲染器

const renderer = new THREE.WebGLRenderer({

  // 由于weapp-adapter会自动创建一个全屏的canvas所以这里直接用

  canvas

})

// 创建场景

const scene = new THREE.Scene()

/**

 * OrthographicCamera是正交相机，

 * 在这种投影模式下，无论物体距离相机距离远或者近，在最终渲染的图片中物体的大小都保持不变。

*/

const camera = new THREE.OrthographicCamera(-width / 2, width / 2, height / 2, -height / 2, 0, 1000)

// 画一个三角形

const triangleShape = new THREE.Shape()

triangleShape.moveTo(0, 100)  // 三角形起始位置

triangleShape.lineTo(-100, -100)

triangleShape.lineTo(100, -100)

triangleShape.lineTo(0, 100)

// 将三角形变成组成物体的几何体

const geometry = new THREE.ShapeGeometry(triangleShape)

// 物体的材料

const material = new THREE.MeshBasicMaterial({

  color: new THREE.Color('#7fffd4'),  // 颜色信息

  side: THREE.DoubleSide         // 用于确定渲染哪一面，因为是旋转的，所以需要正反面都渲染，也就是两面

})

// 组成物体并添加到场景中

const mesh = new THREE.Mesh(geometry, material)

mesh.position.set(0, 0, -200)  // 设置物体在场景中的位置

scene.add(mesh)

camera.position.set(0, 0, 0)  // 相机位置

camera.lookAt(new THREE.Vector3(0, 0, -200)) // 让相机从0, 0, 0 看向 0, 0, -200

renderer.setClearColor(new THREE.Color('#f84462')) // 设置背景色

renderer.setSize(width, height) // 设置最终渲染的尺寸

const render = () => {

  mesh.rotateY(0.05)  // three.js 中旋转角度是通过弧度计算的，公式：度＝弧度×180°/π

  renderer.render(scene, camera)

  requestAnimationFrame(render)

}

render()

```
