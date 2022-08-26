<script setup lang="ts">
// import gsap from 'gsap'
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
} from 'three'
import { Sizes } from '~/interfaces/config.interface'

const canvas = ref<HTMLCanvasElement>()
const sizes = ref<Sizes>({ width: 0, height: 0 })

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

  const scene = new Scene()

  const material = new MeshToonMaterial({
    color: '913854',
    gradientMap: gradientTexture,
  })

  const mesh = new Mesh(new TorusGeometry(1, 0.4, 16, 60), material)

  mesh.position.y = 0
  mesh.position.x = 0

  const directionalLight = createLights()
  const [cameraGroup, camera] = createCamera()

  scene.add(mesh)
  scene.add(directionalLight)
  scene.add(cameraGroup)

  const renderer = new WebGLRenderer({
    canvas: canvas.value,
    alpha: true,
  })
  renderer.setSize(sizes.value.width, sizes.value.height)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))

  const clock = new Clock()
  let previousTime = 0

  const tick = () => {
    const elapsedTime = clock.getElapsedTime()
    const deltaTime = elapsedTime - previousTime
    previousTime = elapsedTime

    // Animate camera
    // cameraGroup.position.y = (-scrollY / sizes.height) * objectDistance

    // const parallaxX = cursor.x
    // const parallaxY = -cursor.y
    // camera.position.x += (parallaxX - camera.position.x) * deltaTime
    // camera.position.y += (parallaxY - camera.position.y) * deltaTime

    // Animate meshed
    // for (const mesh of sectionMeshes) {
    //   mesh.rotation.x += deltaTime * 0.1
    //   mesh.rotation.y += deltaTime * 0.12
    // }

    // Render
    renderer.render(scene, camera)

    // Call tick again on the next frame
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

// /**
//  * Objects
//  */
// const objectDistance = 4

// const material = new THREE.MeshToonMaterial({
//   color: parameters.materialColor,
//   gradientMap: gradientTexture
// })

// const mesh1 = new THREE.Mesh(new THREE.TorusGeometry(1, 0.4, 16, 60), material)
// const mesh2 = new THREE.Mesh(new THREE.ConeGeometry(1, 2, 32), material)
// const mesh3 = new THREE.Mesh(new THREE.TorusKnotGeometry(0.8, 0.35, 100, 16), material)

// mesh1.position.y = -objectDistance * 0
// mesh2.position.y = -objectDistance * 1
// mesh3.position.y = -objectDistance * 2

// mesh1.position.x = 2
// mesh2.position.x = -2
// mesh3.position.x = 2

// scene.add(mesh1, mesh2, mesh3)

// const sectionMeshes = [mesh1, mesh2, mesh3]

// /**
//  * Particles
//  */
// // Geomerty
// const particlesCount = 1000
// const positions = new Float32Array(particlesCount * 3)

// for (let i = 0; i < particlesCount; i++) {
//   positions[i * 3] = (Math.random() - 0.5) * 10
//   positions[i * 3 + 1] =
//     objectDistance * 0.5 - Math.random() * objectDistance * sectionMeshes.length
//   positions[i * 3 + 2] = (Math.random() - 0.5) * 10
// }

// const particlesGeometry = new THREE.BufferGeometry()
// particlesGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))

// const particlesMaterial = new THREE.PointsMaterial({
//   color: parameters.materialColor,
//   sizeAttenuation: true,
//   size: 0.03
// })

// const points = new THREE.Points(particlesGeometry, particlesMaterial)
// scene.add(points)

// window.addEventListener('resize', () => {
//   // Update sizes
//   sizes.width = window.innerWidth
//   sizes.height = window.innerHeight

//   // Update camera
//   camera.aspect = sizes.width / sizes.height
//   camera.updateProjectionMatrix()

//   // Update renderer
//   renderer.setSize(sizes.width, sizes.height)
//   renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
// })

// /**
//  * Scroll
//  */
// let scrollY = window.scrollY
// let currentSection = 0

// window.addEventListener('scroll', () => {
//   scrollY = window.scrollY
//   const newSection = Math.round(scrollY / sizes.height)

//   if (newSection !== currentSection) {
//     currentSection = newSection

//     gsap.to(sectionMeshes[currentSection].rotation, {
//       duration: 1.5,
//       ease: 'power2.inOut',
//       x: '+=6',
//       y: '+=3',
//       z: '+=1.5'
//     })
//   }
// })

// /**
//  * Cursor
//  */
// const cursor = { x: 0, y: 0 }
// window.addEventListener('mousemove', (event) => {
//   cursor.x = event.clientX / sizes.width - 0.5
//   cursor.y = event.clientY / sizes.height - 0.5
// })
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
