[12-Materials]
  Materials材质 是给几何体的每个可见像素上色的
  shader 着色器
  MeshBasicMaterial
  - 对Material 进行材质处理
    1. const material = new THREE.MeshBasicMaterial({map: ...})
    2. const material = new THREE.MeshBasicMaterial()
       material.map = ...

  - Material 颜色表示
    1. material.color = new THREE.Color('#ff0000')
    2. material.color = new THREE.Color('#f00')
    3. material.color = new THREE.Color('red')
    4. material.color = new THREE.Color('rgb(255, 0, 0)')
    5. material.color = new THREE.Color(0xff0000)

  - wireframe 线框
    material.wireframe = true

  - opacity 透明度
    material.opacity = 0.5
  
  - transparent 透明属性
    material.transparent = true

  - alpha 映射

  - side
    THREE.FrontSide (default)
    THREE.BackSide
    THREE.DoubleSide
  ====================================
    MeshNormalMaterial 网格法线材质
    lighting, reflection, refraction, etc  光照, 反射, 折射
    Debug 法线 normal

    - wireframe 线框结构
    - flatShading 线框平面构造
  ====================================
    MeshMatcapMaterial 网格材质球
    matcap图片     Photoshop    Gimp

  ====================================
    MeshDepthMaterial 网格深度材质
    亮度随进度而变大

  ====================================  
    MeshLambertMaterial 

    MeshPhongMaterial
    - material.shiness = 100          // 高光亮度
    - material.specular = new THREE.Color(0xff0000)  镜面反射色

  ====================================
    MeshToonMaterial

    material.gradientMap = gradientTexture
    const gradientTexture = textureLoader.load('url')
    gradientTexture.minFilter = THREE.NearestFilter 
    gradientTexture.magFilter = THREE.NearestFilter
    gradientTexture.generateMipmaps = false

    material.gradientMap = gradientTexture

  =====================================
    MeshStandardMaterial()
    const material = new THREE.MeshStandardMaterial()
    material.roughness = 0.65
    material.metalness = 0.45
    material.map = ....texture

    material{
      .aoMap
      .displacementMap
      .metalnessMap
      .roughnessMap
      .normalScapeMap
      .alphaMap
    }

  =====================================
    MeshPhysicalMaterial
    PonitsMaterial
    ShaderMaterial   RawShaderMaterial
    Environment  

    // HDR 环境贴图加载器
    const rgbeLoader = new RGBELoader(loaderManager)

    const initThree = () => {
      // HDR
      rgbeLoader.load('/static/12-Materials/grasslands_sunset_4k.hdr', (texture) => {
        texture.mapping = THREE.EquirectangularReflectionMapping
        scene!.environment = texture
      })
    }
=====================================

[13-3D Text]
  typeface获取地址
    1.https://gero3.github.io/facetype.js/
    2. Three.js 自带

  1. load the font
  TextBufferGeometry

  const fontLoader = new FontLoader()

  fontLoader.load(
    '/static/13-3D Text/fonts/helvetiker_regular.typeface.json',
    (font) => {
      const textGeometry = new TextGeometry(
        'Hello Three.js',
        {
          font: font,              // 字体
          size: 0.5,               // 字体大小
          depth: 0.2,              // 字体纵深
          curveSegments: 100,      // 采样密度
          bevelEnabled: true,      // 斜角启用
          bevelThickness: 0.03,    // 斜角厚度
          bevelSize: 0.02,         // 斜角尺寸
          bevelOffset: 0,          // 倒角偏移
          bevelSegments: 5,        // 倒角段数
        }
      )
      const textMaterial = new THREE.MeshBasicMaterial({wireframe: true})
      const text = new THREE.Mesh(textGeometry, textMaterial)
      scene?.add(text)
    }
  )

  2. move the font
    I Using the bounding
    bounding是描述几何体所占据的空间  box/sphere  =>  视锥裁剪

    II 位置移动

    textGeometry.computeBoundingBox()
    textGeometry.translate(
      -(textGeometry.boundingBox.max.x + textGeometry.boundingBox.min.x)*0.5,
      -(textGeometry.boundingBox.max.y + textGeometry.boundingBox.min.y)*0.5,
      -(textGeometry.boundingBox.max.z + textGeometry.boundingBox.min.z)*0.5,
    )

    III 装饰品
    for(let i=0; i<100; i++){
      const donutGeometry = new THREE.TorusGeometry(0.3, 0.2, 20, 45)
      const donut = new THREE.TorusGeometry(0.3, 0.2, 20, 45)

      dount.position. = (Math.random()-0.5)*10
      dount.position. = (Math.random()-0.5)*10
      dount.position. = (Math.random()-0.5)*10
      dount.rotation = Math.random() * Math.PI * 5
      dount.rotation = Math.random() * Math.PI * 5
      dount.rotation = Math.random() * Math.PI * 5
      const scale = Math.random()
      donut.scale.set(scale, scale, scale)
      scene.add(donut)
    }

[14 - Live 网站公开]
  1.  npm run build
  2.  Vercel
  3.  Netlify
  4.  GitHub pages