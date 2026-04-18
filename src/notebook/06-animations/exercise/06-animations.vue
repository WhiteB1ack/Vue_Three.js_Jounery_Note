<template>
  <div ref="threeContainer"></div>
</template>

<script setup lang="ts" name="Animations">  
import * as THREE from 'three'
import {onMounted, onUnmounted, ref} from 'vue'
import gsap from 'gsap'

// 获取元素DOM
const threeContainer = ref<HTMLDivElement | null>(null)

// 全局设置
let renderer: THREE.WebGLRenderer | null = null
let scene: THREE.Scene | null = null
let camera: THREE.PerspectiveCamera | null = null
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
  camera = new THREE.PerspectiveCamera(45, sizes.width/sizes.height)
  camera.position.set(0, 0, 5)
  camera.lookAt(0, 0, 0)
  scene.add(camera)

  // AxesHelper
  const axeshelper = new THREE.AxesHelper(5)
  scene.add(axeshelper)

  // Renderer
  renderer = new THREE.WebGLRenderer()
  renderer.setSize(sizes.width, sizes.height)

  // Time
  //  let time = Date.now()

  // Clock
  //const clock = new THREE.Clock()

  // GSAP
  gsap.to(mesh.position, { duration: 1, delay: 1, x: 2})
  gsap.to(mesh.position, { duration: 1, delay: 2, x: -2})
  gsap.to(camera.position, { duration: 1, delay: 1, x: -2 })
  gsap.to(camera.position, { duration: 1, delay: 2, x: 2})
  
  // Animations
  const tick = () => {
    if(mesh === null || renderer === null || scene === null || camera === null) return

    // Clock
    //const elapsedTime = clock.getElapsedTime()
    //console.log(elapsedTime)

    // Time
    /*
    const currentTime = Date.now()
    const deltaTime = currentTime - time
    time = currentTime*/

    

    // Update object
    //camera.position.y = 2*Math.sin(elapsedTime)
    //camera.position.x = 2*Math.cos(elapsedTime)
    //camera.lookAt(mesh.position)

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