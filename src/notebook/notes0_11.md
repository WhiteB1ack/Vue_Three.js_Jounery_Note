[03-canvas页面渲染]
canvas元素获取渲染

<div ref="threeContainer"></div>

const threeContainer = ref<HTMLDivElement | null>(null)

// Renderer
renderer = new THREE.WebGLRenderer()
renderer.setSize(sizes.width, sizes.height)
renderer.render(scene, camera)

// canvas元素
threeContainer.value?.appendChild(renderer.domElement)

[05-transform objects]
Mesh
  - position
  - scale
  - rotation
  - quaternion

  -- position
    mesh.position.x = 1
    mesh.position.y = 1
    mesh.position.z = 1
    mesh,position.set(x, y, z)

    Vector3向量本质
      length: position.length()
      distance: position1.distanceTo(position2)
      normalize: position.normalize
  -- scale
    mesh.scale.x = 2
    mesh.scale.y = 2
    mesh.scale.z = 2
    mesh.scale.set(2, 2, 2)

  -- rotation(Euler)
    mesh.rotation.reorder('')
    mesh.rotation.x = Math.PI / 2
    mesh.rotation.y = Math.PI / 2
  欧拉角由于自锁性质, 每一次旋转只是相对于上一次坐标, 导致局限性颇大

  -- quaternion

Camera
  - camera.lookAt(Vector3)

Group
  - position
  - group.add(children)

[06-Animating]
  - 实时渲染
  const tick = () => {
    // Render
    renderer.render(scene, camera)
    window.requestAnimationFrame(tick)
  }

  - 控制旋转I
  // Time
  let time = Date.now()
  // Animations
  const tick = () => {
    .....

    const currentTime = Date.now()
    const deltaTime = currentTime - time
    time = currentTime

    // ratation
    mesh.rotation.y += 0.001*deltaTime
  }
  - 控制旋转 II
  // Clock
  const clock = new THREE.Timer();
  timer.connect(document)

  // Animations
  const tick = () => {
    const deltaTime = timer.getDelta()
    const elapsedTime = timer.getElapsedTime()

    mesh.rotation.y = elapsedTime * Math.PI / 2
  }
  >>> 变体 将rotation变为position, 实现物块和摄像机的上升

  mesh.position.x = Math.sin(elapsedTime)
  mesh.position.y = Math.cos(elapsedTime)

  - GSAP
  gsap.to(mesh.position,   { duration: 1, delay: 1, x: 2  })
  gsap.to(mesh.position,   { duration: 1, delay: 2, x: -2 })
  gsap.to(camera.position, { duration: 1, delay: 1, x: -2 })
  gsap.to(camera.position, { duration: 1, delay: 2, x: 2  })
  gsap.to(position,    { duration: time, delay: time, x:  })

[07-camera]
  camera{
    Camera:  基础相机
    PerspectiveCamera:  透视相机
    ArrayCamera: 数组相机
    CubeCamera:  立方相机
    OrthographicCamera: 正交相机     
    StereoCamera: 深景相机
  }
  PerspectiveCamera(fov, aspect ratio, near parameter, far parameter)
    fov: 垂直视野   45~75
    aspect ratio: 宽高比
    near and far parameters共同构成了观测范围
    两者不可过于取极限, 会导致相近图层重叠
  
  OrthographicCamera(left, right, top, bottom, near, far)
    定义矩形区域内, 失去透视效果的图示

  摇杆相机I
  // Cursor
  const cursor = {
    x: 0,
    y: 0,
  }
  window.addEventListener('mousemove', (event) => {
    cursor.x = event.clientX / sizes.width - 0.5
    cursor.y = event.clientY / sizes.height - 0.5
  })

  const tick = () => {
    // Update camera
    camera.position.x = cursor.x * 3
    camera.position.y = cursor.y * 3
  }
  >>> 注: 由于事件客户端默认Y是向下的, 而Three.js默认Y是向上的

  摇杆相机II
  window.addEventListener('mousemove', (event) => {
    cursor.x = event.clientX / sizes.width - 0.5
    cursor.y = -(event.clientY / sizes.height - 0.5)
  })

  const tick = () => {
    camera.position.x = Math.sin(cursor.x * Math.PI * 2) * 3
    camera.position.z = Math.cos(cursor.x * Math.PI * 2) * 3
    camera.position.y = cursor.y * 5
    camera.lookAt(mesh.position) 
  }

  [controls]
    DeviceOrientationControls    // 移动端   Error
    DragControls
    FirstPersonControls
    FlyControls
    OrbitCOntrols
    PointerLockControls ***
    TrackballControls
    TransformControls

  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

  // Controls
  const controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.target.y = 2

  // Animations
  const tick = () => {
    // Update controls
    controls.update()
  }

[08-fullscreen]
  // Sizes
  const sizes = {
    width: window.innerWidth
    height: 
  }

  window.addEventListener('resize', () => {
    if(!renderer) return

    // Update sizes
    sizes.width = window.innerWidth
    sizes.height = window.innerHeight

    // Update camera
    camera.aspect = sizes.width / sizes.height
    camera.updateProjectionMatrix()

    // Update renderer
    renderer.setSize(sizes.width, sizes.height)
  })

  window.addEventListener('dblclick', () => {
    if(!document.fullscreenElement){
      renderer?.domElement.requestFullscreen()
    }else{
      document.exitFullscreen()
    }
  })

[09-Geometry]、
  Define:  面 <三角形>
  
  - overview
    - BoxGeometry
    - PlaneGeometry
    - CircleGeometry
    - ConeGeometry
    - CylinderGeometry
    - RingGeometry
    - TorusGeometry
    - TorusKnotGeometry
    - DodecahedromGeometry
    ....

  BoxGeometry(width, height, depth, widthSegments, heightSegments, depthSegments)
  三角形代表一个面, 画布渲染的三角形(平面)越多, 画布细节越多

  创建缓冲几何体
  Float32Array : 类型数组, 仅限浮点数, 长度固定, 易处理
  // Float32Array
  # way 1
  const positionsArray = new Float32Array(9)

  positionsArray[0] = 0
  positionsArray[1] = 0
  positionsArray[2] = 0

  positionsArray[3] = 0
  positionsArray[4] = 1
  positionsArray[5] = 0
  
  positionsArray[6] = 1
  positionsArray[7] = 0
  positionsArray[8] = 0

  # way 2
  const positionsArray = new Float32Array([
    0, 0, 0,
    0, 1, 0,
    1, 0, 0
  ])

  // convert Float32Array to BufferAttribute(缓冲属性)
  const positionsAttribute = new THREE.BufferAttribute(positionsArray, 3)

  // geometry   create 缓冲几何体
  const geometry = new THREE.BufferGeometry()
  geometry.setAttribute('position', positionsAttribute)

  Float32Array -> BufferAttribute  
                +                     ->  geometry
           BufferGeometry    

  // 变体
  const geometry = new THREE.BufferGeometry()

  const count = 50
  const positionsArray = new Float32Array(count * 3 * 3)

  // random num
  for(let i=0; i<count*3*3; i++){
    positionsArray[i] = (Math.random() - 0.5) * 4
  }

  const positionsAttribute = new THREE.BufferAttribute(positionsArray, 3)
  geometry.setAttribute('position', positionsAttribute)      

  // 拓展 index 索引调用节点

[10 - DebugUI]
  dat GUI

  Range - [minimum ... maximum]
    gui.add(mesh.position, 'y', minimum, maximum, step)
    gui
      .add(mesh.position, 'y')
      .min(minimum)
      .max(maximum)
      .step(step)
      .name('name')

  Color - [#000000 ... #ffffff]
    gui.addColor(material, color)

  Text - simple text
  const params = {
    message: 'Hello Three.js'
  }
  gui.add(params, 'message')

  Checkbox - booleans (true or false)
    gui.add(mesh, 'visible')
  Select - a choice from list

  Button - trigger function
  const action = {
    spin: () => {  gsap.to(mesh.rotation, { duration: 1, y: mesh.position.y + 10 })   }
  }
  gui.add(action, 'spin')


  Folder - organize panel
  const cubeFolder = gui.addFolder('cube controls')
  cubeFolder.add(mesh.scale, 'x', 1, 10)
  cubeFolder.add(mesh.scale, 'y', 1, 10)
  cubeFolder.add(mesh.scale, 'z', 1, 10)

[11 - Texture]
  纹理以图像为基础
  图像覆盖在几何体表面上
  COLOR(OR ALBEDO)  
  ALPHA
  HEIGHT(OR DISPLACEMENT)
  NORMAL
  AMBIENT OCCLUSION
  METALNESS
  ROUGHNESS

  << PBR >>  基于物理渲染, 贴近现实生活的算法计算
  如何添加纹理{
    1. 获取图片url
      - 将image置于src内, 并import导入
      - 图片置于/static/文件夹 access导入  http://localhost:5173/static/color.jpg
    2. 加载图片     image -> texture -> (map)
      --- 方法I
        ```
        const image = new Image()
        const texture = new THREE.Texture(image)

        image.onload = () => {
          texture.needUpdate = true
        }

        material = new THREE.MeshBasicMaterial({ map: texture })
      
      ```
      --- 方法II
      ```
        const textureLoader = new THREE.TextureLoader()
        const texture = textureLoader.load(
          '/static/color.jpg',
          () => {
            // load
            console.log('load')
          },
          () => {
            // progress
            console.log('process')
          },
          () => {
            // error
            console.log('error')
          }
        )
      ```
      --- 方法III
      const loadingManager = new THREE.LoadingManager()

      loadingManager.onStart = () => {
        console.log('onStart')
      }
      loadingManager.onLoad = () => {
        console.log('onLoad')
      }
      loadingManager.onProcess = () => {
        console.log('onProcess')
      }
      loadingManager.onError = () => {
        console.log('Error')
      }

      const textureLoader = new THREE.TextureLoader(loadingManager)
      const colorTexture = textureLoader.load('/static/color.jpg')
      const heightTexture = textureLoader.load('/static/height.jpg')
      const normalTexture = textureLoader.load('/static/normal.jpg')
      ....
    3. 纹理处理
      重复次数
      colorTexture.repeat.x = 2   
      colorTexture.repeat.y = 3
      
      环绕方式
      colorTexture.wrapS = THREE.RepeatWraping
      colorTexture.wrapT = THREE.RepeatWraping

      偏移
      colorTexture.offset.x = 0.5
      colorTexture.offset.y = 0.5

      旋转
      colorTexture.rotation = Math.PI / 4  

      旋转中心
      colorTexture.center.x = 0.5
      colorTexture.center.y = 0.5

      filtering 过滤

            MIP 映射 mapping
      colorTexture.generateMipmaps = false          // 保留原始尺寸
      colorTexture.minFilter = THREE.NearestFilter  // 纹理缩小时, 最近邻过滤
      colorTexture.magFilter = THREE.NearestFilter  // 纹理放大时, 最近邻过滤


      摩尔纹?????

      纹理的格式和优化
      { widget, size, data } TintPng   压缩
      1:jpg 糊但轻 2: png  清但重  3: basis

      尽可能小  尺寸为2**n  2**n

      法线纹理: 精确!!!!   png   lossless

      合适的纹理与格式



      poliigon.com
      3dtextures.me
      arroway-textures.ch
  }