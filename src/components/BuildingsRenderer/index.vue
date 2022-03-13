<template>
  <div ref="rootRef" class="buildings-renderer"></div>
</template>

<script>
import { reactive, toRefs, defineComponent, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { FBXLoader } from 'three/examples/jsm/loaders/FBXLoader'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
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
      const point = new THREE.PointLight(0xffffff)
      point.position.set(500, 500, 500)
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

      // 创建渲染器对象
      const renderer = new THREE.WebGLRenderer()
      threeState.renderer = renderer
      renderer.setSize(width, height) // 设置渲染区域尺寸
      renderer.setClearColor(0xb9d3ff, 1) // 设置背景颜色
      state.rootRef.appendChild(renderer.domElement)

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
        //mesh.rotateY(0.001)
        //requestAnimationFrame(render)
        stats.end()
      }
      threeState.render = render
      render()

      // 鼠标控制
      const controls = new OrbitControls(camera, renderer.domElement)
      threeState.controls = controls
      controls.addEventListener('change', render)

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
      const intersectObjects = raycaster.intersectObjects(threeState.scene.children, true)
      // 最近的物体
      const firstIntersectObject = intersectObjects.length ? intersectObjects[0] : null
      console.log('intersectObjects', intersectObjects)
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