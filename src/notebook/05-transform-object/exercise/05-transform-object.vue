<template>
  <div ref="threeContainer"></div>
</template>

<script setup lang="ts" name="transform_object">
import * as THREE from 'three'
import {ref, onMounted, onUnmounted} from 'vue'

// 获取元素dom
const threeContainer = ref<HTMLDivElement | null>(null)

// 全局设置
let renderer: THREE.WebGLRenderer | null = null
let scene: THREE.Scene | null = null
let camera: THREE.PerspectiveCamera | null = null
let mesh: THREE.Mesh | null = null

const initThree = () => {
  // Scene
  scene = new THREE.Scene()

  // Group
  const group = new THREE.Group()
  group.position.set(-2, -2, -2)

  // Red Cube
  const geometry = new THREE.BoxGeometry(1, 1, 1)
  const material = new THREE.MeshBasicMaterial({color: 0xff0000})
  mesh = new THREE.Mesh(geometry, material)
  mesh.position.y = 1
  mesh.position.x = 1
  mesh.position.z = 1
  mesh.position.set(-10, -10, -10)

  mesh.scale.set(1, 1, 1)
  console.log(mesh.position.length())
  // normalize    length -> 1
  mesh.position.normalize()
  console.log(mesh.position.length())

  // Rotation
  mesh.rotation.reorder('XYZ')
  mesh.rotation.x = Math.PI / 2
  mesh.rotation.y = Math.PI / 2
  mesh.rotation.z = Math.PI / 2


  // Group
  const cube1 = new THREE.Mesh(
    new THREE.BoxGeometry(1, 1, 1),
    new THREE.MeshBasicMaterial({color:0xff0000})
  )
  cube1.position.x = 2
  group.add(cube1)

  const cube2 = new THREE.Mesh(
    new THREE.BoxGeometry(1, 1, 1),
    new THREE.MeshBasicMaterial({color:0x00ff00})
  )
  cube2.position.y = 2
  group.add(cube2)

  
  const cube3 = new THREE.Mesh(
    new THREE.BoxGeometry(1, 1, 1),
    new THREE.MeshBasicMaterial({color:0x0000ff})
  )
  cube3.position.z = 2
  group.add(cube3)


  scene.add(group)
  scene.add(mesh)

  // Sizes
  const sizes = {
    width: 800,
    height: 600
  }

  // Camera
  camera = new THREE.PerspectiveCamera(75, sizes.width/sizes.height)
  camera.position.set(3, 3, 3)
  camera.lookAt(mesh.position)
  scene.add(camera)

  console.log(mesh.position.distanceTo(camera.position))

  // AxesHelper
  const axeshelper = new THREE.AxesHelper(5)
  scene.add(axeshelper)

  // Renderer
  renderer = new THREE.WebGLRenderer()
  renderer.setSize(sizes.width, sizes.height)
  renderer.render(scene, camera)

  // canvas元素
  threeContainer.value?.appendChild(renderer.domElement)
}

onMounted(()=> {
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