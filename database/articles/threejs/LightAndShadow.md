### 一. 修改物体材质

接着上一篇的项目，在上一篇中物体的材质都是用的 MeshBasicMaterial 这种材质，这种材质是不受光照的影响的，所以要修改成 MeshPhongMaterial 这种材质，让它受光照的影响。

在 cylinder.js 和 box.js 中将 MeshBasicMaterial 改为 MeshPhongMaterial

```
const material = new THREE.MeshPhongMaterial({
    color: 0xffffff
})
```

这时候就渲染结果就是这个样子：

![](/madao.github.io/database/images/articles/threejs/light_and_shadow/image.png)

物体都看不见了，因为没有光

### 二. 添加环境光

官方文档是这样解释环境光的：

> 环境光会均匀的照亮场景中的所有物体。环境光不能用来投射阴影，因为它没有方向。

可以理解为从各个方向照向物体的光

在 renderer 目录下，新建 light.js

```
const light = () => {
  return {
    ambientLight: new THREE.AmbientLight(0xffffff, 0.8)
  }
}
export default light
```

`new THREE.AmbientLight`第一个参数是光的颜色，第二个参数是光强。

scene.js

```
import light from './light'

const scene = () => {
  const sceneInstance = new THREE.Scene()
  const axesHelper = new THREE.AxesHelper(100)
  const lights = light()
  Object.keys(lights).forEach(value => {
    sceneInstance.add(lights[value])
  })
  sceneInstance.add(axesHelper)
  return sceneInstance
}

export default scene
```

效果：

![](/madao.github.io/database/images/articles/threejs/light_and_shadow/image1.png)

现在物体就可以看见了，由于环境光不能投射阴影，所以需要其他的光，下面试下其他光的效果

### 三. 其他类型的光

- #### 平行光

  平行光就是从光源处到物体的光线都是平行的，官方文档上说类似太阳光，它是可以产生阴影的。

  light.js

  ```
  const light = () => {
      const directionalLight = new THREE.DirectionalLight(0xFF8C00)
      directionalLight.position.set(10, 30, 20) // 设置光源的位置
      return {
          ambientLight: new THREE.AmbientLight(0xffffff, 0.5),
          directionalLight
      }
  }
  export default light
  ```

  看下效果：

  ![](/madao.github.io/database/images/articles/threejs/light_and_shadow/image2.png)

  因为不加环境光太暗了，所以还是在有环境光的情况下测试，这时候就能看出来物体不同的面受到的光照强度是不同的。

- #### 半球光

  并没有从文档上看懂半球光是什么。。先记住一点半球光不能产生阴影，把之前的平行光改成半球光：

  light.js

  ```
  const light = () => {
      const hemisphereLight = new THREE.HemisphereLight(0x8A2BE2, 0xFFFAFA, 1)
      hemisphereLight.position.set(10, 30, 20) // 设置光源的位置
      return {
          ambientLight: new THREE.AmbientLight(0xffffff, 0.5),
          hemisphereLight
      }
  }
  export default light
  ```

  效果：

  ![](/madao.github.io/database/images/articles/threejs/light_and_shadow/image3.png)

  这篇文章给了详细的例子，可以看看。[地址](https://www.jianshu.com/p/4cfab4e3ec3a)

- #### 点光源

  这个好理解，就像蜡烛或者灯泡发出的光线。

  light.js

  ```
  const light = () => {
      const pointLight = new THREE.PointLight(0x1a94bc, 1, 100)
      pointLight.position.set(10, 30, 20) // 设置光源的位置
      return {
          ambientLight: new THREE.AmbientLight(0xffffff, 0.5),
          pointLight
      }
  }
  export default light
  ```

  看起来和平行光也很像

  ![](/madao.github.io/database/images/articles/threejs/light_and_shadow/image4.png)

  但是这个点光源是有一个衰减值的，就是光源离物体越远，光的强度就越低，平行光不会，距离再远也是相同的光强

- #### 平面光光源

  这个从名字上就能理解，就是从一个平面发出平行光。

  light.js

  ```
  const light = () => {
      const pointLight = new THREE.PointLight(0x1a94bc, 1, 100, 100)
      pointLight.position.set(10, 30, 20) // 设置光源的位置
      return {
          ambientLight: new THREE.AmbientLight(0xffffff, 0.5),
          pointLight
      }
  }
  export default light
  ```

  这种光只支持 MeshStandardMaterial 和 MeshPhysicalMaterial 两种材质，所以还得去改一下 box.js 和 cylinder.js 中物体的材质

  ```
  const material = new THREE.MeshStandardMaterial({
    color: 0xffffff
  })
  ```

  和平行光的效果差不多

  ![](/madao.github.io/database/images/articles/threejs/light_and_shadow/image5.png)

- #### 聚光灯

  这个在官方文档上有个很好的例子，一看就明白了，这里就不写了。

  看了效果之后决定选择平行光，因为能产生阴影，效果也还可以，最终 light.js 的代码：

  ```
  const light = () => {
      const directionalLight = new THREE.DirectionalLight(0xffffff, 0.3)
      directionalLight.position.set(10, 30, 20) // 设置光源的位置
      return {
          ambientLight: new THREE.AmbientLight(0xffffff, 0.8),
          directionalLight
      }
  }
  export default light
  ```

  记得把 box.js 和 cylinder.js 中的材质改回来（MeshPhongMaterial）

### 三. 实现阴影效果

实现阴影效果需要具备以下条件：

1. 渲染器启用阴影
2. 可以形成阴影的光源
3. 能够表现阴影的材质
4. 物体可以投射阴影
5. 接受阴影的物体

现在就一步一步实现上面的条件：

- #### 渲染器开启阴影

  renderer -> index.js 添加

  ```
  init () {
      this.instance.shadowMap.enabled = true   // 新增，渲染器启用阴影
  }
  ```

- #### 形成阴影的光源

  上面说过平行光是可以投射阴影的，所以用它就好了。

  ligth.js

  ```
  const light = () => {
      const directionalLight = new THREE.DirectionalLight(0xffffff, 0.3)
      directionalLight.position.set(10, 30, 20) // 设置光源的位置
      directionalLight.castShadow = true  // 新增，产生动态阴影

      directionalLight.shadow.camera.near = 0.5 // 新增
      directionalLight.shadow.camera.far = 500 // 新增
      directionalLight.shadow.camera.left = -100 // 新增
      directionalLight.shadow.camera.right = 100 // 新增
      directionalLight.shadow.camera.top = 100 // 新增
      directionalLight.shadow.camera.bottom = -100 // 新增

      directionalLight.shadow.mapSize.width = 1024 // 新增
      directionalLight.shadow.mapSize.height = 1024 // 新增
      return {
          ambientLight: new THREE.AmbientLight(0xffffff, 0.8),
          directionalLight
      }
  }
  export default light
  ```

- #### 能够表现阴影的材质

  物体使用的是 MeshPhongMaterial 材质，这个材质是可以产生阴影的，这里就不用改了

- #### 物体可以投射阴影

  在 cylinder.js 和 box.js 中分别添加如下代码：

  ```
  create () {
      this.instance.castShadow = true // 新增，让物体可以投射阴影
      this.instance.receiveShadow = true // 新增，让物体可以接受阴影
  }
  ```

- 接受阴影的物体

  在现实生活中接受阴影的物体最常见的就是地面了，这里也是一样，需要给现在的场景添加一个地面

  新建 ground.js
  ![](/madao.github.io/database/images/articles/threejs/light_and_shadow/image6.png)

  ```
  const ground = () => {
      const groundGeometry = new THREE.PlaneGeometry(200, 200)
      const groundMaterial = new THREE.MeshStandardMaterial({
          color: 0xffffff
      })
      const instance = new THREE.Mesh(groundGeometry, groundMaterial)
      instance.receiveShadow = true

      return instance
  }

  export default ground
  ```

  现在去看看效果：

  ![](/madao.github.io/database/images/articles/threejs/light_and_shadow/image7.png)

  这个效果炸了呀。

  ground 变成了墙而不是地面。所以需要旋转一下

  让他沿着 x 轴转负 90 度

  ```
  instance.rotateX(-Math.PI / 2)
  ```

  效果：

  !![](/madao.github.io/database/images/articles/threejs/light_and_shadow/image8.png)

  现在就和 x 轴在同一平面了，只是有点高，把物体切断了，再往下移一点

  ```
  instance.position.y = -5
  ```

  现在的效果是：

  ![](/madao.github.io/database/images/articles/threejs/light_and_shadow/image9.png)

- ### 添加背景

  由于这样的地面太丑，现在不用 MeshStandardMaterial 这个材质了，改用 ShadowMaterial 材质，[文档](https://threejs.org/docs/index.html#api/zh/materials/ShadowMaterial)

  ground.js

  ```
  const ground = () => {
      const groundGeometry = new THREE.PlaneGeometry(200, 200)
      const groundMaterial = new THREE.ShadowMaterial({
          color: 0x000000,
          opacity: 0.3
      })
      const instance = new THREE.Mesh(groundGeometry, groundMaterial)
      instance.rotateX(-Math.PI / 2)
      instance.position.y = -5
      instance.receiveShadow = true

      return instance
  }

  export default ground

  ```

  改完之后现在整个地面又变黑了，暂时不用管，加上背景就好了

  背景应该和相机看到的范围一样大，所以背景的尺寸要用到相机相关的变量 size，这里为了方便维护，把这个变量单独存放：

  ![](/madao.github.io/database/images/articles/threejs/light_and_shadow/image10.png)

  camera_config.js

  ```
  export default {
      size: 30
  }
  ```

  新建 background.js

  ![](/madao.github.io/database/images/articles/threejs/light_and_shadow/image11.png)

  ```
  import cameraConfig from '../../configs/camera_config'

  const background = () => {
      const backgroundGeometry = new THREE.PlaneGeometry(
          cameraConfig.size * 2,
          window.innerHeight / window.innerWidth * cameraConfig.size * 2
      )
      const backgroundMaterial = new THREE.MeshBasicMaterial({
          color: 0xd7dbe6
      })
      const instance = new THREE.Mesh(backgroundGeometry, backgroundMaterial)
      instance.position.z = -85

      return instance
  }

  export default background

  ```

  camera.js 修改为下面这样

  ```
  import cameraConfig from '../../configs/camera_config'  // 新增

  const camera = () => {
      const ratio = window.innerHeight / window.innerWidth
      const { size } = cameraConfig  // 改动
      // 设置相机可看到范围
      const cameraInstance = new THREE.OrthographicCamera(
          -size, size, size * ratio, -size * ratio, -100, 85  // 改动
      )

      cameraInstance.position.set(-10, 10, 10) // 设置相机位置
      cameraInstance.up.set(0, 1, 0)
      cameraInstance.lookAt(new THREE.Vector3(0, 0, 0)) // 设置相机位置从0, 0, 0望向0, 0, 0

      return cameraInstance
  }

  export default camera

  ```

  renderer -> index.js

  ```
  import camera from './camera'
  import scene from './scene'
  import background from '../objects/background'  // 新增

  class Renderer {
      constructor () {
          this.camera = null
          this.scene = null
      }
      init () {
          this.camera = camera()
          this.scene = scene()
          const { width, height } = canvas
          if (window.devicePixelRatio) {
              canvas.height = height * window.devicePixelRatio
              canvas.width = width * window.devicePixelRatio
          }
          this.instance = new THREE.WebGLRenderer({
              canvas,
              antialias: true
          })
          this.instance.shadowMap.enabled = true
          this.scene.add(this.camera)  // 新增，这一步很重要
          this.camera.add(background())  // 新增
      }

      render () {
          this.instance.render(this.scene, this.camera)
      }
  }

  export default new Renderer()

  ```

  去开发者工具看看效果：

  ![](/madao.github.io/database/images/articles/threejs/light_and_shadow/image12.png)

  现在就比较正常了。

  完整代码[GitHub](https://github.com/GreedyWhale/finger_mourning/tree/lightAndShadow)

接下来就是实现跳一跳中的小人了
