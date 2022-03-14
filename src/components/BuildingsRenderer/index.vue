<template>
  <div ref="rootRef" class="buildings-renderer"></div>
</template>

<script>
import { reactive, toRefs, defineComponent, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { FBXLoader } from 'three/examples/jsm/loaders/FBXLoader'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer'
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass'
import { OutlinePass } from 'three/examples/jsm/postprocessing/OutlinePass'
import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass'
import { FXAAShader } from "three/examples/jsm/shaders/FXAAShader.js"

import Stats from 'three/examples/jsm/libs/stats.module'

export default defineComponent({
  setup (props, ctx) {
    const state = reactive({
      rootRef: null,
    })

    const threeState = {
      scene: null,
      camera: null,
      renderer: null,
      controls: null,
      render: () => {},
      //intersectedObject: null,
      composer: null
    }
    const highlightIntersects = intersects => {
      if (!intersects.length) {
        return threeState.composer = null
      }
      const findBuildingObject = object => {
        //if (object.name === 'mergedBlocks') return object
        //if (object.parent) return findBuildingObject(object.parent)
        //return null
        return object
      }

      const { clientWidth: width, clientHeight: height } = state.rootRef
      const { scene, renderer, camera } = threeState
      // 最近的物体
      const firstIntersectObject = intersects.length ? intersects[0].object : null
      const buildingObject = findBuildingObject(firstIntersectObject)
      const selectedObjects = [buildingObject]
      // 特效组件
      const composer = new EffectComposer(renderer)
      // 新建一个场景通道  为了覆盖到原理来的场景上
      const renderPass = new RenderPass(scene, camera)
      composer.addPass(renderPass)
      // 物体边缘发光通道
      const outlinePass = new OutlinePass(new THREE.Vector2(width, height), scene, camera)
      outlinePass.selectedObjects = selectedObjects
      outlinePass.edgeStrength = 10.0 // 边框的亮度
      outlinePass.edgeGlow = 1 // 光晕[0,1]
      outlinePass.usePatternTexture = false // 是否使用父级的材质
      outlinePass.edgeThickness = 1.0 // 边框宽度
      outlinePass.downSampleRatio = 2 // 边框弯曲度
      outlinePass.pulsePeriod = 5 // 呼吸闪烁的速度
      outlinePass.visibleEdgeColor.set(parseInt(0x00ff00)) // 呼吸显示的颜色
      outlinePass.hiddenEdgeColor = new THREE.Color(0, 0, 0) // 呼吸消失的颜色
      outlinePass.clear = true
      composer.addPass(outlinePass)
      // 自定义的着色器通道 作为参数
      const effectFXAA = new ShaderPass(FXAAShader)
      effectFXAA.uniforms.resolution.value.set(1 / width, 1 / height)
      effectFXAA.renderToScreen = true
      composer.addPass(effectFXAA)
      threeState.composer = composer
      threeState.render()
    }

    const initThree = () => {
      // 创建场景
      const scene = new THREE.Scene()
      threeState.scene = scene

      // 创建网格模型
      //const geometry = new THREE.BoxGeometry(100, 100, 100)
      //const material = new THREE.MeshLambertMaterial({
      //  color: 0x0000ff
      //})
      //const mesh = new THREE.Mesh(geometry, material)
      //scene.add(mesh)

      // 创建地面
      //const planeBufferGeometry = new THREE.PlaneBufferGeometry(10, 10);
      //const plane = new THREE.Mesh(planeBufferGeometry, new THREE.MeshLambertMaterial({color: 0xFFFFFF, side: THREE.DoubleSide}))
      //plane.rotation.x = -Math.PI / 2
      //scene.add(plane)

      // 添加网格线
      //scene.add(new THREE.GridHelper(10, 10))

      // 光源设置
      //点光源
      const point = new THREE.PointLight(0xFFFFFF)
      point.position.set(500, 500, 500)
      // https://threejs.org/docs/index.html#api/zh/lights/shadows/PointLightShadow
      point.castShadow = true // default false
      scene.add(point)
      //环境光
      const ambient = new THREE.AmbientLight(0xFFFFFF)
      scene.add(ambient)

      // 相机设置
      const { clientWidth: width, clientHeight: height } = state.rootRef
      const k = width / height // 窗口宽高比
      const s = 30 // 三维场景显示范围控制系数，系数越大，显示范围越大
      const camera = new THREE.OrthographicCamera(-s * k, s * k, s, -s, 1, 1000)
      threeState.camera = camera
      camera.position.set(500, 500, 500) // 设置相机位置
      camera.lookAt(scene.position) // 设置相机方向(指向的场景对象)
      //{ // camera helper
      //  const helper = new THREE.CameraHelper( camera )
      //  scene.add( helper )
      //}

      // 创建渲染器对象
      const renderer = new THREE.WebGLRenderer()
      threeState.renderer = renderer
      renderer.setSize(width, height) // 设置渲染区域尺寸
      renderer.setClearColor(0xb9d3ff, 1) // 设置背景颜色
      state.rootRef.appendChild(renderer.domElement)

      // 鼠标控制
      const controls = new OrbitControls(camera, renderer.domElement)
      threeState.controls = controls

      const animationMixer = new THREE.AnimationMixer(scene)
      const clock  = new THREE.Clock()
      // 显示帧数据
      const stats = Stats()
      state.rootRef.appendChild(stats.dom)
      const render = () => {
        stats.begin()
        animationMixer.update(clock.getDelta())
        // 执行渲染操作
        renderer.render(scene, camera)
        if (threeState.composer) {
          threeState.composer.render()
        }
        //mesh.rotateY(0.001)
        //requestAnimationFrame(render)
        stats.end()
      }
      render()
      controls.addEventListener('change', render)
      threeState.render = render

      // load model
      //fbx
      //const loader = new FBXLoader()
      //loader.load('/threeModels/buildings/fbx/buildings.fbx', fbx => {
      //  console.log('fbx', fbx)
      //  scene.add(fbx)
      //  render()
      //}, xhr => {
      //  console.info((xhr.loaded / xhr.total) * 100 + '% loaded')
      //}, err => {
      //  console.error(err)
      //})

      const loader = new GLTFLoader()
      //loader.load('/threeModels/gltf/Soldier.glb', gltf => {
      //  console.log('gltf', gltf)
      //  scene.add(gltf.scene)
      //  const walkAnimation = gltf.animations.find(i => i.name === 'Walk')
      //  const action = animationMixer.clipAction(walkAnimation)
      //  action.play()
      //  render()
      //}, xhr => {
      //  console.info((xhr.loaded / xhr.total) * 100 + '% loaded')
      //}, err => {
      //  console.error(err)
      //})
      loader.load('/threeModels/gltf/buildings/scene.gltf', gltf => {
        console.log('gltf', gltf)
        gltf.scene.name = 'buildings'
        scene.add(gltf.scene)
        render()
      }, xhr => {
        console.info((xhr.loaded / xhr.total) * 100 + '% loaded')
      }, err => {
        console.error(err)
      })
    }

    const onWindowResize = () => {
      const { clientWidth: width, clientHeight: height } = state.rootRef
      threeState.camera.aspect = width / height
      threeState.camera.updateProjectionMatrix()
      threeState.renderer.setSize(width, height)
      threeState.render()
    }

    const onRendererDomClick = (event) => {
      const { offsetX, offsetY } = event
      const { clientWidth: width, clientHeight: height } = state.rootRef
      const x = (offsetX / width) * 2 - 1
      const y = -(offsetY / height) * 2 + 1
      const mousePosition = new THREE.Vector2(x, y)

      const raycaster = new THREE.Raycaster()
      // 坐标在相机的位置
      raycaster.setFromCamera(mousePosition, threeState.camera)
      // 检测哪些物体被点击
      const intersects = raycaster.intersectObjects(threeState.scene.children, true)
      console.log('intersects', intersects)
      highlightIntersects(intersects)
      //if (threeState.intersectedObject) {
      //  threeState.intersectedObject.material.color.set(threeState.intersectedObject.originHexColor)
      //  threeState.intersectedObject = null
      //}
      //最近的物体
      //const firstIntersectObject = intersects.length ? intersects[0].object : null
      //if (firstIntersectObject) {
      //  if (firstIntersectObject !== threeState.intersectedObject) {
      //    firstIntersectObject.originHexColor = firstIntersectObject.originHexColor || firstIntersectObject.material.color.getHex()
      //    firstIntersectObject.material.color.set(0x66ff00)
      //    threeState.intersectedObject = firstIntersectObject
      //  }
      //  threeState.render()
      //}
    }

    onMounted(() => {
      initThree()
      window.addEventListener('resize', onWindowResize)
      threeState.renderer.domElement.addEventListener('click', onRendererDomClick)
    })
    onUnmounted(() => {
      window.removeEventListener('resize', onWindowResize)
      threeState.renderer.domElement.removeEventListener('click', onRendererDomClick)
    })
    return { ...toRefs(state) }
  },
})
</script>

<style scoped>
.buildings-renderer {
  width: 100%;
  height: 100%;
}
</style>