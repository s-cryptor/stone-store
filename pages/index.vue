<script setup lang="ts">
import gsap from 'gsap'
import {
  NearestFilter,
  TextureLoader,
  Scene,
  MeshToonMaterial,
  Mesh,
  TorusGeometry,
  DirectionalLight,
  Group,
  PerspectiveCamera,
  WebGLRenderer,
  Clock,
  ConeGeometry,
  TorusKnotGeometry,
  BufferGeometry,
  BufferAttribute,
  PointsMaterial,
  Points,
} from 'three'
import { Sizes } from '~/interfaces/config.interface'

const canvas = ref<HTMLCanvasElement>()
const sizes = ref<Sizes>({ width: 0, height: 0 })

const scene = new Scene()

onBeforeMount(() => {
  updateSizes()
  window.addEventListener('resize', onResizeWindow)
})

onMounted(() => {
  render()
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', onResizeWindow)
})

function onResizeWindow(): void {
  updateSizes()
}

function updateSizes(): void {
  sizes.value.width = window.innerWidth
  sizes.value.height = window.innerHeight
}

function render(): void {
  const textureLoader = new TextureLoader()
  const gradientTexture = textureLoader.load('/assets/textures/gradients/3.jpg')
  gradientTexture.magFilter = NearestFilter

  const objectDistance = 4

  const material = new MeshToonMaterial({
    color: '#913854',
    gradientMap: gradientTexture,
  })

  const mesh1 = new Mesh(new TorusGeometry(1, 0.4, 16, 60), material)

  mesh1.position.y = -objectDistance * 0
  mesh1.position.x = 2

  scene.add(mesh1)

  const sectionMeshes = [mesh1]

  const particlesCount = 1000
  const positions = new Float32Array(particlesCount * 3)

  for (let i = 0; i < particlesCount; i++) {
    positions[i * 3] = (Math.random() - 0.5) * 10
    positions[i * 3 + 1] =
      objectDistance * 0.5 - Math.random() * objectDistance * sectionMeshes.length
    positions[i * 3 + 2] = (Math.random() - 0.5) * 10
  }

  const particlesGeometry = new BufferGeometry()
  particlesGeometry.setAttribute('position', new BufferAttribute(positions, 3))

  const particlesMaterial = new PointsMaterial({
    color: '#fabc77',
    sizeAttenuation: true,
    size: 0.03,
  })

  const points = new Points(particlesGeometry, particlesMaterial)
  scene.add(points)

  const directionalLight = createLights()
  const [cameraGroup, camera] = createCamera()

  scene.add(directionalLight)
  scene.add(cameraGroup)

  const renderer = new WebGLRenderer({
    canvas: canvas.value,
    alpha: true,
  })
  renderer.setSize(sizes.value.width, sizes.value.height)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))

  let scrollY = window.scrollY
  let currentSection = 0

  window.addEventListener('scroll', () => {
    scrollY = window.scrollY
    const newSection = Math.round(scrollY / sizes.value.height)

    if (newSection !== currentSection) {
      currentSection = newSection

      // gsap.to(sectionMeshes[currentSection].rotation, {
      //   duration: 1.5,
      //   ease: 'power2.inOut',
      //   x: '+=6',
      //   y: '+=3',
      //   z: '+=1.5',
      // })
    }
  })

  window.addEventListener('resize', () => {
    // Update camera
    camera.aspect = sizes.value.width / sizes.value.height
    camera.updateProjectionMatrix()

    // Update renderer
    renderer.setSize(sizes.value.width, sizes.value.height)
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  })

  const cursor = { x: 0, y: 0 }
  window.addEventListener('mousemove', (event) => {
    cursor.x = event.clientX / sizes.value.width - 0.5
    cursor.y = event.clientY / sizes.value.height - 0.5
  })

  const clock = new Clock()
  let previousTime = 0

  const tick = () => {
    const elapsedTime = clock.getElapsedTime()
    const deltaTime = elapsedTime - previousTime
    previousTime = elapsedTime

    cameraGroup.position.y = (-scrollY / sizes.value.height) * objectDistance

    const parallaxX = cursor.x
    const parallaxY = -cursor.y
    camera.position.x += (parallaxX - camera.position.x) * deltaTime
    camera.position.y += (parallaxY - camera.position.y) * deltaTime

    for (const mesh of sectionMeshes) {
      mesh.rotation.x += deltaTime * 0.1
      mesh.rotation.y += deltaTime * 0.12
    }

    renderer.render(scene, camera)

    window.requestAnimationFrame(tick)
  }

  tick()
}

function createLights(): DirectionalLight {
  const directionalLight = new DirectionalLight('#ffffff', 1)
  directionalLight.position.set(1, 1, 0)
  return directionalLight
}

function createCamera(): [Group, PerspectiveCamera] {
  const cameraGroup = new Group()
  const camera = new PerspectiveCamera(35, sizes.value.width / sizes.value.height, 0.1, 100)
  camera.position.z = 6
  cameraGroup.add(camera)

  return [cameraGroup, camera]
}
</script>

<template>
  <div>
    <canvas ref="canvas" class="webgl"></canvas>

    <section class="section">
      <h1>My Portfolio</h1>
    </section>
    <section class="section">
      <h2>My projects</h2>
    </section>
    <section class="section">
      <h2>Contact me</h2>
    </section>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
}

html {
  background-color: #1e1a20;
}

.webgl {
  position: fixed;
  top: 0;
  left: 0;
  outline: none;
}

.section {
  display: flex;
  align-items: center;
  height: 100vh;
  position: relative;
  font-family: 'Cabin', sans-serif;
  color: #ffeded;
  text-transform: uppercase;
  font-size: 7vmin;
  padding-left: 10%;
  padding-right: 10%;
}

section:nth-child(odd) {
  justify-content: flex-end;
}
</style>
