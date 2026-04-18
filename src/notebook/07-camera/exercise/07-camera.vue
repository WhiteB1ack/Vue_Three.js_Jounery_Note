<template>
  <div ref="threeContainer"></div>
</template>

<script setup lang="ts" name="Camera">
import * as THREE from 'three'
import {onMounted, onUnmounted, ref} from 'vue'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

// 获取元素 DOM
const threeContainer = ref<HTMLDivElement | null>(null)

// 全局设置
let renderer: THREE.WebGLRenderer | null = null
let scene: THREE.Scene | null = null
let mesh: THREE.Mesh | null = null

const initThree = () => {
  // Scene
  scene = new THREE.Scene()

  // Red Cube
  const geometry = new THREE.BoxGeometry(1, 1, 1)
  const material = new THREE.MeshBasicMaterial({color: 0xff0000})
  mesh = new THREE.Mesh(geometry, material)  
  
  scene.add(mesh)

  // Sizes
  const sizes = {
    width: 800,
    height: 600
  }

  // Camera
  const camera = new THREE.PerspectiveCamera(45, sizes.width/sizes.height)

  // const aspectRatio = sizes.width / sizes.width
  // const camera = new THREE.OrthographicCamera(-1 * aspectRatio, 1 * aspectRatio, 1, -1, 0.1, 100)
  // camera.position.set(2, 2, 2)
  camera.position.z = 10
  camera.lookAt(mesh.position)
  scene.add(camera)

  // AxesHelper
  // const axeshelper = new THREE.AxesHelper(5)
  // scene.add(axeshelper)

  //Cursor
  const cursor = { 
    x: 0,
    y: 0,
  }
  window.addEventListener('mousemove', (event) => {
    cursor.x = event.clientX / sizes.width - 0.5
    cursor.y = -(event.clientY / sizes.height - 0.5)
  })

  // Renderer
  renderer = new THREE.WebGLRenderer()
  renderer.setSize(sizes.width, sizes.height)

  // Controls
  const controls = new OrbitControls(camera, renderer?.domElement)
  controls.enableDamping = true
  controls.target.y = 2

  // Animations
  const tick = () => {
    if(mesh === null || renderer === null || scene === null || camera === null) return

    // Rotation
    // mesh.rotation.y += 0.01

    // Update camera
    // camera.position.x = cursor.x * 10
    // camera.position.y = cursor.y * 10
    // camera.position.x = Math.sin(cursor.x * Math.PI * 2) * 3
    // camera.position.z = Math.cos(cursor.x * Math.PI * 2) * 3
    // camera.position.y = cursor.y * 5
    // camera.lookAt(mesh.position)

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