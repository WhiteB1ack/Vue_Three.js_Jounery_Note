<template>
  <div ref="threeContainer" class="container"></div>
</template>

<script setup lang="ts" name="threeD_Text">
import {ref, onMounted, onUnmounted} from 'vue'
import * as THREE from 'three'
import { OrbitControls, Wireframe } from 'three/examples/jsm/Addons.js';
import * as dat from 'lil-gui'
import { FontLoader } from 'three/examples/jsm/Addons.js';
import { TextGeometry } from 'three/examples/jsm/Addons.js';

// 获取元素DOM
const threeContainer = ref<HTMLDivElement | null>(null)

// 全局设置
let renderer: THREE.WebGLRenderer | null = null
let scene: THREE.Scene | null = null
let mesh: THREE.Mesh | null = null

// Texture
const textureLoader = new THREE.TextureLoader()
const matcapTextutre = textureLoader.load('/static/color1.jpg')

// const texture = textureLoader.load()

// Fonts
const fontLoader = new FontLoader()

fontLoader.load(
  '/static/13-3D-Text/fonts/helvetiker_regular.typeface.json',
  (font) => {
    const textGeometry = new TextGeometry(
      'Hello Three.js',
      {
        font: font,          // 字体
        size: 0.5,           // 字体大小          
        depth: 0.01,          // 字体纵深          
        curveSegments: 5,  // 采样密度          
        bevelEnabled: true,  // 斜角启用          
        bevelThickness: 0.03,// 斜角厚度          
        bevelSize: 0.02,     // 斜角尺寸          
        bevelOffset: 0,      // 倒角偏移          
        bevelSegments: 4,    // 倒角段数           
      }
    )
    // textGeometry.computeBoundingBox()
    // if(textGeometry.boundingBox){
    //   textGeometry.translate(
    //     -(textGeometry.boundingBox.max.x + textGeometry.boundingBox.min.x)*0.5,
    //     -(textGeometry.boundingBox.max.y + textGeometry.boundingBox.min.y)*0.5,
    //     -(textGeometry.boundingBox.max.z + textGeometry.boundingBox.min.z)*0.5,
    //   )  
    // }

    textGeometry.center()
    console.log(textGeometry.boundingBox)
    const material = new THREE.MeshBasicMaterial({map: matcapTextutre})
    const text = new THREE.Mesh(textGeometry, material)
    scene?.add(text)

    console.time('donuts')

    const donutGeometry = new THREE.TorusGeometry(0.3, 0.2, 20, 45)

    for(let i = 0; i<1000; i++){

      const donut = new THREE.Mesh(donutGeometry, material)

      donut.position.x = (Math.random()-0.5)* 10
      donut.position.y = (Math.random()-0.5)* 10
      donut.position.z = (Math.random()-0.5)* 10

      donut.rotation.x = Math.random()* Math.PI* 5
      donut.rotation.y = Math.random()* Math.PI* 5
      donut.rotation.z = Math.random()* Math.PI* 5

      const scale = Math.random()
      donut.scale.set(scale, scale, scale)

      scene?.add(donut)
    }
    console.time('donuts')
  }
)

const initThree = () => {
  // Scene
  scene = new THREE.Scene

  // Red Cube
  const geometry = new THREE.BoxGeometry(1, 1, 1)
  const material = new THREE.MeshBasicMaterial({color: 0xff0000})
  mesh = new THREE.Mesh(geometry, material)
  scene.add(mesh)

  mesh.visible = false

  // Sizes
  const sizes = {
    width: window.innerWidth,
    height: window.innerHeight
  }

  // Camera
  const camera = new THREE.PerspectiveCamera(45, sizes.width/sizes.height)

  camera.position.set(0, 0, 10)
  camera.lookAt(0, 0, 0)
  scene.add(camera)
  
  // Debug GUI
  const gui = new dat.GUI()

  // Axes helper
  // const axeshelper = new THREE.AxesHelper()
  // scene.add(axeshelper)

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

  // Animations
  const tick = () => {
    if(!mesh || !renderer || !scene || !camera) return

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