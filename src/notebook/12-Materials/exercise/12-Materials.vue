<template>
  <div ref="threeContainer" class="container"></div>
</template>

<script setup lang="ts" name="Materials">
import {ref, onMounted, onUnmounted} from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/Addons.js';
import { pl } from 'element-plus/es/locale/index.mjs';
import * as dat from 'lil-gui'
import { metalness } from 'three/tsl';
import { RGBELoader } from 'three/examples/jsm/Addons.js';

// 获取元素DOM
const threeContainer = ref<HTMLDivElement | null>(null)

// 全局设置
let renderer: THREE.WebGLRenderer | null = null
let scene: THREE.Scene | null = null

// Texture
const loadingManager = new THREE.LoadingManager()
const textureLoader = new THREE.TextureLoader(loadingManager)
const spheretexture = textureLoader.load('/static/color1.jpg')
const planetexture = textureLoader.load('/static/color2.jpg')
const torustexture = textureLoader.load('/static/color3.jpg')
const matcaptexture = textureLoader.load('/static/matcap2.png')
const gradientTexture = textureLoader.load('https://cs.wellesley.edu/~cs307/threejs/r124/three.js-master/examples/textures/gradientMaps/threeTone.jpg')
const aoTexture = textureLoader.load('/static/12-Materials/forest_ground_04_ao_4k.jpg')
const displacetexture = textureLoader.load('/static/12-Materials/forest_ground_04_disp_4k.png')
const normaltexture = textureLoader.load('/static/12-Materials/forest_ground_04_nor_gl_4k.exr')

// HDR 环境贴图加载器
const rgbeLoader = new RGBELoader(loadingManager)

gradientTexture.minFilter = THREE.NearestFilter
gradientTexture.magFilter = THREE.NearestFilter
gradientTexture.generateMipmaps = false


const initThree = () => {
  // Scene
  scene = new THREE.Scene

  // Objects
  // const material = new THREE.MeshNormalMaterial({flatShading: true})
  //const spherematerial = new THREE.MeshBasicMaterial({map: spheretexture, color: '0xff0000', opacity: 0.5, transparent: true})
  // const material = new THREE.MeshDepthMaterial()
  // const material = new THREE.MeshLambertMaterial({color: '0x000000'})
  // const material = new THREE.MeshPhongMaterial({shininess: 100, specular: new THREE.Color(0xff0000)})

  // const material = new THREE.MeshStandardMaterial()
  // material.roughness = 0.65
  // material.metalness = 0.45
  // material.map = matcaptexture
  // material.aoMap = aoTexture
  // material.aoMapIntensity = 10
  // material.displacementMap = displacetexture
  // material.displacementScale = 0.5
  // material.normalMap = normaltexture
  // material.normalScale.set(0.1, 0.1)
  const material = new THREE.MeshStandardMaterial()
  material.metalness = 0.7
  material.roughness = 0.2
  // material.envMap = rgbetexture  // 移到下面 HDR 加载回调里设置

  const sphere = new THREE.Mesh(
    new THREE.SphereGeometry(0.5, 64, 64),
    material
  )
  sphere.position.x = -1.5
  if(sphere.geometry.attributes.uv){
    sphere.geometry.setAttribute('uv2', sphere.geometry.attributes.uv?.clone())
  }

  //const planematerial = new THREE.MeshBasicMaterial({map: planetexture, color: '0x00ff00', opacity: 0.5, transparent: true})
  const plane = new THREE.Mesh(
    new THREE.PlaneGeometry(1, 1, 100, 100),
    material
  )
  console.log(plane.geometry.attributes.uv?.array)

  // const geometry = plane.geometry
  // if (geometry.attributes.uv) {
  //     geometry.setAttribute('uv2', geometry.attributes.uv.clone())
  // } 

  if (plane.geometry.attributes.uv) {
    plane.geometry.setAttribute('uv2', plane.geometry.attributes.uv.clone());
  }

  //const toursmaterial = new THREE.MeshBasicMaterial({map: torustexture, color: '0x0000ff', opacity: 0.5, transparent: true})
  const torus = new THREE.Mesh(
    new THREE.TorusGeometry(0.3, 0.2, 64, 128),
    material
  )
  torus.position.x = 1.5
  if(torus.geometry.attributes.uv){
    torus.geometry.setAttribute('uv2', torus.geometry.attributes.uv.clone())
  }

  scene.add(sphere, plane, torus)

  // Lights
  const ambientLight = new THREE.AmbientLight(0xffffff, 5)
  scene.add(ambientLight)

  const pointLight = new THREE.PointLight(0xffffff, 1.0)
  pointLight.position.set(5, 3, 4)
  scene.add(pointLight)

  // Sizes
  const sizes = {
    width: window.innerWidth,
    height: window.innerHeight
  }

  // Camera
  const camera = new THREE.PerspectiveCamera(45, sizes.width/sizes.height)

  camera.position.z = -10

  camera.lookAt(0, 0, 0)
  scene.add(camera)

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

  // Debug GUI
  const gui = new dat.GUI()
  gui.add(material, 'roughness', 0, 1, 0.01)
  gui.add(material, 'metalness', 0, 1, 0.02)
  // gui.addColor(material, 'color')
  // gui.add(material, 'aoMapIntensity', 0, 10, 0.1)
  // gui.add(material, 'wireframe')
  // gui.add(material, 'displacementScale', 0, 5, 0.05)
  gui.add(camera.position, 'x', -100, 100)
  gui.add(camera.position, 'y', -100, 100)
  gui.add(camera.position, 'z', -100, 100)

  // Cursor
  const cursor = {
    x: 0,
    y: 0,
  }
  window.addEventListener('mousemove', (event) => {
    cursor.x = event.clientX / sizes.width - 0.5
    cursor.y = -(event.clientX / sizes.height - 0.5)
  })

  // Renderer
  renderer = new THREE.WebGLRenderer()
  renderer.setSize(sizes.width, sizes.height)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))

  // Controls
  const controls = new OrbitControls(camera, renderer?.domElement)
  controls.enabled = true
  controls.enableDamping = true
  controls.target.set(0, 2, 0)

  // ✅ HDR 环境贴图异步加载 - 必须在 renderer 创建后加载
  rgbeLoader.load('/static/12-Materials/grasslands_sunset_4k.hdr', (texture) => {
    texture.mapping = THREE.EquirectangularReflectionMapping
    
    // 方式1：设置场景环境（推荐，所有 MeshStandardMaterial 自动使用）
    scene!.environment = texture
    
    // 方式2：也可以直接赋给材质（和上面二选一即可）
    // material.envMap = texture
    // material.needsUpdate = true
  })

  const timer = new THREE.Timer()
  timer.connect(document)

  // Animations
  const tick = () => {
    if( !renderer || !scene || !camera) return
    timer.update()
    const deltaTime = timer.getDelta()
    const elapsedTime = timer.getElapsed()

    // Update objects
    // sphere.rotation.y = 0.1 * elapsedTime
    // plane.rotation.y = 0.1 * elapsedTime
    // torus.rotation.y = 0.1 * elapsedTime

    // sphere.rotation.x = 0.1 * elapsedTime
    // plane.rotation.x = 0.1 * elapsedTime
    // torus.rotation.x = 0.1 * elapsedTime

    // Update controls
    controls.update()

    // Render
    renderer.render(scene, camera)
    window.requestAnimationFrame(tick)
  }
  tick()

  // canvas 元素
  threeContainer.value?.appendChild(renderer.domElement)  
}

onMounted(() => {
  if(!threeContainer.value) return
  initThree()
})

onUnmounted(() => {
  if(renderer){
    renderer.dispose()
    renderer = null
  }
})

</script>

<style>
* {
  margin: 0;
  padding: 0;
}

body {
  overflow: hidden;
}

.container {
  position: fixed;
  top: 0;
  left: 0;
  outline: none
}
</style>