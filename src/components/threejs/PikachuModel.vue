  <template>
    <div ref="sceneContainer" class="model-container"></div>
  </template>

  <script lang="ts" setup>
  import { onMounted, ref, onBeforeUnmount } from 'vue'
  import * as THREE from 'three'
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
  import type { GLTF } from 'three/examples/jsm/loaders/GLTFLoader.js'

  const sceneContainer = ref<HTMLDivElement | null>(null)

  onMounted(() => {
    if (!sceneContainer.value) return

    const scene = new THREE.Scene()
    const clock = new THREE.Clock()

    const camera = new THREE.PerspectiveCamera(
      75,
      sceneContainer.value.clientWidth / sceneContainer.value.clientHeight,
      0.1,
      1000
    )
    camera.position.set(0.02, 0.1, 0.9)
    camera.rotation.x = -Math.PI / 7

    const renderer = new THREE.WebGLRenderer({ alpha: true, antialias: true })
    renderer.setSize(sceneContainer.value.clientWidth, sceneContainer.value.clientHeight)
    renderer.shadowMap.enabled = true
    renderer.shadowMap.type = THREE.PCFSoftShadowMap
    sceneContainer.value.appendChild(renderer.domElement)

    const ambientLight = new THREE.AmbientLight(0xffffff, 0.7)
    scene.add(ambientLight)

    const directionalLight = new THREE.DirectionalLight(0xffffff, 2)
    directionalLight.position.set(5, 10, 5)
    directionalLight.castShadow = true
    directionalLight.shadow.mapSize.width = 2048
    directionalLight.shadow.mapSize.height = 2048
    directionalLight.shadow.camera.near = 1
    directionalLight.shadow.camera.far = 20

    // Soften shadow edges to make it look more natural
    directionalLight.shadow.bias = -0.01
    directionalLight.shadow.radius = 4

    scene.add(directionalLight)

    const floorGeometry = new THREE.PlaneGeometry(10, 10)
    const floorMaterial = new THREE.ShadowMaterial({ opacity: 0.3 })
    const floor = new THREE.Mesh(floorGeometry, floorMaterial)
    floor.rotation.x = -Math.PI / 2
    floor.position.y = -1
    floor.receiveShadow = true
    scene.add(floor)

    const loader = new GLTFLoader()
    let mixer: THREE.AnimationMixer
    let model: THREE.Object3D | null = null

    loader.load('/models/pikachu.glb', (gltf: GLTF) => {
      model = gltf.scene
      model.traverse((child: THREE.Object3D) => {
        if ((child as THREE.Mesh).isMesh) {
          const mesh = child as THREE.Mesh
          mesh.castShadow = true
          mesh.receiveShadow = true
        }
      })
      scene.add(model)
      model.position.set(0, -1, 0)

      mixer = new THREE.AnimationMixer(model)

      const idleClip = gltf.animations.find(clip => clip.name === 'Idle')
      if (idleClip) {
        const action = mixer.clipAction(idleClip)
        action.play()
      } else {
        console.warn('Idle animation not found.')
      }
    })

    // Rotate model with mouse
    let isDragging = false
    let previousMouseX = 0

    const onMouseDown = (event: MouseEvent) => {
      isDragging = true
      previousMouseX = event.clientX
    }

    // limit the rotation
    const MIN_ROTATION_Y = -0.3
    const MAX_ROTATION_Y = 0.3

    const onMouseMove = (event: MouseEvent) => {
      if (!isDragging || !model) return
      const deltaX = event.clientX - previousMouseX
      // sensitivity
      const rotationDelta = deltaX * 0.01
      const newRotationY = THREE.MathUtils.clamp(model.rotation.y + rotationDelta, MIN_ROTATION_Y, MAX_ROTATION_Y)
      model.rotation.y = newRotationY
      previousMouseX = event.clientX
    }

    const onMouseUp = () => {
      isDragging = false
    }

    sceneContainer.value.addEventListener('mousedown', onMouseDown)
    window.addEventListener('mousemove', onMouseMove)
    window.addEventListener('mouseup', onMouseUp)

    const animate = () => {
      requestAnimationFrame(animate)
      const delta = clock.getDelta()
      if (mixer) mixer.update(delta)
      renderer.render(scene, camera)
    }

    animate()

    window.addEventListener('resize', () => {
      if (!sceneContainer.value) return
      camera.aspect = sceneContainer.value.clientWidth / sceneContainer.value.clientHeight
      camera.updateProjectionMatrix()
      renderer.setSize(sceneContainer.value.clientWidth, sceneContainer.value.clientHeight)
    })

    onBeforeUnmount(() => {
      sceneContainer.value?.removeEventListener('mousedown', onMouseDown)
      window.removeEventListener('mousemove', onMouseMove)
      window.removeEventListener('mouseup', onMouseUp)
    })
  })
  </script>

  <style scoped>
  .model-container {
    width: 10%;
    height: 400px;
  }
  </style>