<template>
  <div ref="threeContainer" class="container"></div>
</template>

<script setup lang="ts" name="DebugUI">
import {ref, onMounted, onUnmounted} from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/Addons.js';

// 获取元素DOM
const threeContainer = ref<HTMLDivElement | null>(null)

// 全局设置
let renderer: THREE.WebGLRenderer | null = null
let scene: THREE.Scene | null = null
let mesh: THREE.Mesh | null = null

const initThree = () => {
  // Scene
  scene = new THREE.Scene

  // Red Cube
  const geometry = new THREE.BoxGeometry(1, 1, 1)
  const material = new THREE.MeshBasicMaterial({color: 0xff0000})
  mesh = new THREE.Mesh(geometry, material)
  scene.add(mesh)

  // Sizes
  const sizes = {
    width: window.innerWidth,
    height: window.innerHeight
  }

  // Camera
  const camera = new THREE.PerspectiveCamera(45, sizes.width/sizes.height)

  camera.position.z = 10
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