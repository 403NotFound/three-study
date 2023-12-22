<script setup>
import { ref, onMounted, h } from "vue"
import * as THREE from "three"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js"
import { GUI } from "three/examples/jsm/libs/lil-gui.module.min.js"
// 引入gltf模型加载库GLTFLoader.js
import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js"
// import Stat from 'three/examples/jsm/libs/stats.module.js'

const canvasRef = ref(null)
onMounted(() => {
  handleCreateThreeWorld()
})
let moveDirection = {
  forward: false,
  backward: false,
  left: false,
  right: false,
}
const handleInitScene = () => {
  const scene = new THREE.Scene()
  // scene.background = new THREE.Color(0xffffff)
  return scene
}

const handleInitCamera = () => {
  const camera = new THREE.PerspectiveCamera(
    45,
    window.innerWidth / window.innerHeight,
    1,
    3000
  )
  camera.position.set(0, 0, -10)
  camera.lookAt(10, 0, 0)
  return camera
}

const handleInitObject = () => {
  const geometry = new THREE.BoxGeometry(50, 50, 50)
  const material = new THREE.MeshPhongMaterial({
    color: 0x0000ff,
  })
  const mesh = new THREE.Mesh(geometry, material)
  return mesh
}

const handleInitGLTFLoader = scene => {
  const loader = new GLTFLoader()
  loader.load(
    "/brother.glb",
    function (gltf) {
      console.log("场景3D模型树结构", gltf.scene)
      scene.add(gltf.scene)
    },
    function (xhr) {
      // 控制台查看加载进度xhr
      // 通过加载进度xhr可以控制前端进度条进度
      const percent = xhr.loaded / xhr.total
      console.log("加载进度" + percent)
    }
  )
}

const handleInitPointLight = () => {
  const pointLight = new THREE.PointLight(0xffffff, 1)
  // 亮度
  pointLight.intensity = 1000
  pointLight.position.set(10, 10, 10)

  // 加一个环境光
  return pointLight
}

const handleInitPointLightHelper = pointLight => {
  const pointLightHelper = new THREE.PointLightHelper(pointLight, 10)
  return pointLightHelper
}

const handleInitAxesHelper = () => {
  const axesHelper = new THREE.AxesHelper(250)
  return axesHelper
}

const handelInitRenderer = (width, height) => {
  const renderer = new THREE.WebGLRenderer()
  renderer.setSize(width, height)
  return renderer
}

const handelInitOrbitControls = (camera, dom) => {
  const controls = new OrbitControls(camera, dom)
  return controls
}

const handleCreateThreeWorld = () => {
  // 创建场景
  const width = window.innerWidth
  const height = window.innerHeight

  const scene = handleInitScene()
  // 创建相机
  const camera = handleInitCamera()
  // 模型对象

  // const mesh = handleInitObject()

  const gltf = handleInitGLTFLoader(scene)
  // handleCreatGUI(mesh)

  // 点光源
  const pointLight = handleInitPointLight()
  scene.add(pointLight)

  // 光源辅助观察
  const pointLightHelper = handleInitPointLightHelper(pointLight)
  scene.add(pointLightHelper)

  // AxesHelper：辅助观察的坐标系
  const axesHelper = handleInitAxesHelper()
  scene.add(axesHelper)

  // 创建渲染器
  const renderer = handelInitRenderer(width, height)

  // 设置相机控件轨道控制器OrbitControls
  const controls = handelInitOrbitControls(camera, renderer.domElement)
  // 将渲染器的dom元素添加到canvasRef中
  canvasRef.value.appendChild(renderer.domElement)
  // 渲染函数
  function render() {
    if (moveDirection.forward) {
      camera.position.z -= 0.1
    }
    if (moveDirection.backward) {
      camera.position.z += 0.1
    }
    if (moveDirection.left) {
      camera.position.x -= 0.1
    }
    if (moveDirection.right) {
      camera.position.x += 0.1
    }
    renderer.render(scene, camera) //执行渲染操作
    controls.update() // 添加这行代码
    requestAnimationFrame(render) //请求再次执行渲染函数render，渲染下一帧
  }
  render()
  renderer.render(scene, camera)
  renderer.outputEncoding = THREE.sRGBEncoding
  // onresize 事件会在窗口被调整大小时发生
  window.onresize = function () {
    // 重置渲染器输出画布canvas尺寸
    renderer.setSize(window.innerWidth, window.innerHeight)
    // 全屏情况下：设置观察范围长宽比aspect为窗口宽高比
    camera.aspect = window.innerWidth / window.innerHeight
    // 渲染器执行render方法的时候会读取相机对象的投影矩阵属性projectionMatrix
    // 但是不会每渲染一帧，就通过相机的属性计算投影矩阵(节约计算资源)
    // 如果相机的一些属性发生了变化，需要执行updateProjectionMatrix ()方法更新相机的投影矩阵
    camera.updateProjectionMatrix()
  }

  window.addEventListener("keydown", function (event) {
    switch (event.key) {
      case "w":
        moveDirection.forward = true
        break
      case "s":
        moveDirection.backward = true
        break
      case "a":
        moveDirection.left = true
        break
      case "d":
        moveDirection.right = true
        break
    }
  })

  window.addEventListener("keyup", function (event) {
    switch (event.key) {
      case "w":
        moveDirection.forward = false
        break
      case "s":
        moveDirection.backward = false
        break
      case "a":
        moveDirection.left = false
        break
      case "d":
        moveDirection.right = false
        break
    }
  })
}

const handleCreatGUI = mesh => {
  // 实例化一个gui对象
  const gui = new GUI()
  //改变交互界面style属性
  gui.domElement.style.right = "0px"
  gui.domElement.style.width = "300px"

  gui.add(mesh.position, "x", 0, 500)
  gui.add(mesh.position, "y", 0, 500)
  gui.add(mesh.position, "z", 0, 500)
  // gui.add(pointLight, "intensity", 0, 2000000).name("环境光强度")
}
</script>

<template>
  <div ref="canvasRef"></div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
