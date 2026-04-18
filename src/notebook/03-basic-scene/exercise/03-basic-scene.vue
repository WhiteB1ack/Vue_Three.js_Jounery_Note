<template>
  <div ref="threeContainer"></div>
</template>

<script setup lang="ts" name="basic_scene">
import * as THREE from 'three'

import { onMounted, onUnmounted, ref } from 'vue'

// 获取容器元素的引用
const threeContainer = ref<HTMLDivElement | null>(null)

let renderer: THREE.WebGLRenderer | null = null
let scene: THREE.Scene | null = null
let camera: THREE.PerspectiveCamera | null = null
let mesh: THREE.Mesh | null = null
let animationId: number | null = null

const initThree = () => {
  // Scene
  scene = new THREE.Scene()

  // Red Cube
  const geometry = new THREE.BoxGeometry(1, 1, 1)
  const material = new THREE.MeshBasicMaterial({ color: 0xff0000 })
  mesh = new THREE.Mesh(geometry, material)

  scene.add(mesh)

  // Sizes
  const sizes = {
    width: 800,
    height: 600
  }

  // Camera
  camera = new THREE.PerspectiveCamera(75, sizes.width/sizes.height);
  camera.position.set(3, 3, 3)
  camera.lookAt(0, 0, 0)
  scene.add(camera)

  // Renderer
  renderer = new THREE.WebGLRenderer()
  renderer.setSize( sizes.width, sizes.height)
  renderer.render(scene, camera)

  // canvas元素
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