<template>
  <div ref="threeContainer" class="container"></div>
</template>

<script setup lang="ts" name="Texture">
import {ref, onMounted, onUnmounted} from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/Addons.js';

// 获取元素DOM
const threeContainer = ref<HTMLDivElement | null>(null)

// 全局设置
let renderer: THREE.WebGLRenderer | null = null
let scene: THREE.Scene | null = null
let mesh: THREE.Mesh | null = null

const loadingManager = new THREE.LoadingManager()

loadingManager.onStart = () => {
  console.log('onStart')
}
loadingManager.onLoad = () => {
  console.log('onLoad')
}
loadingManager.onProgress = () => {
  console.log('onProgress')
}
loadingManager.onError = () => {
  console.log('onError')
}

const textureLoader = new THREE.TextureLoader(loadingManager)          // 一个可以加载多个, 函数型
const colorTexture = textureLoader.load('/static/color.jpg',)

// colorTexture.repeat.x = 2
// colorTexture.repeat.y = 3
// colorTexture.wrapS = THREE.RepeatWrapping
// colorTexture.wrapT = THREE.RepeatWrapping

// colorTexture.offset.x = 0.5
// colorTexture.offset.y = 0.5

// colorTexture.rotation = Math.PI/4
// colorTexture.center.x = 0.5
// colorTexture.center.y = 0.5 
colorTexture.generateMipmaps = false
colorTexture.minFilter = THREE.NearestFilter
colorTexture.magFilter = THREE.NearestFilter

const initThree = () => {
  // Scene
  scene = new THREE.Scene

  // Red Cube
  const geometry = new THREE.BoxGeometry(1, 1, 1)
  console.log(geometry.attributes.uv)
  const material = new THREE.MeshBasicMaterial({
    map: colorTexture
  })
  mesh = new THREE.Mesh(geometry, material)
  scene.add(mesh)

  // Sizes
  const sizes = {
    width: window.innerWidth,
    height: window.innerHeight
  }

  // Camera
  const camera = new THREE.PerspectiveCamera(45, sizes.width/sizes.height)

  camera.position.z = 5
  camera.lookAt(mesh.position)
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
  controls.target.set(0, 0, 0)

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