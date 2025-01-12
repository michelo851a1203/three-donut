<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'
// import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { OrbitControls } from '@three-ts/orbit-controls';

const canvas = ref<HTMLCanvasElement | null>(null)

onMounted(() => {
  if (!canvas.value) return

  const scene = new THREE.Scene()
  const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  const renderer = new THREE.WebGLRenderer({ canvas: canvas.value, antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.shadowMap.enabled = true
  renderer.shadowMap.type = THREE.PCFSoftShadowMap

  const controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05

  const geometry = new THREE.TorusGeometry(1, 0.4, 32, 100)
  
  const uniforms = {
    time: { value: 0 },
    color1: { value: new THREE.Color('#69ff9d') },
    color2: { value: new THREE.Color('#2ecc71') },
  }

  const material = new THREE.ShaderMaterial({
    uniforms: uniforms,
    vertexShader: `
      varying vec2 vUv;
      varying vec3 vNormal;
      void main() {
        vUv = uv;
        vNormal = normal;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
      }
    `,
    fragmentShader: `
      uniform float time;
      uniform vec3 color1;
      uniform vec3 color2;
      varying vec2 vUv;
      varying vec3 vNormal;
      void main() {
        vec3 light = normalize(vec3(1.0, 1.0, 1.0));
        float d = dot(vNormal, light) * 0.5 + 0.5;
        vec3 color = mix(color1, color2, vUv.x + sin(time) * 0.3);
        gl_FragColor = vec4(color * d, 1.0);
      }
    `
  })

  const donut = new THREE.Mesh(geometry, material)
  donut.castShadow = true
  scene.add(donut)

  const planeGeometry = new THREE.PlaneGeometry(10, 10)
  const planeMaterial = new THREE.ShadowMaterial({ opacity: 0.5 })
  const plane = new THREE.Mesh(planeGeometry, planeMaterial)
  plane.rotation.x = -Math.PI / 2
  plane.position.y = -2
  plane.receiveShadow = true
  scene.add(plane)

  const ambientLight = new THREE.AmbientLight(0xffffff, 0.2)
  scene.add(ambientLight)

  const pointLight = new THREE.PointLight(0xffffff, 1.5)
  pointLight.position.set(5, 5, 5)
  pointLight.castShadow = true
  pointLight.shadow.mapSize.width = 1024
  pointLight.shadow.mapSize.height = 1024
  scene.add(pointLight)

  camera.position.z = 5

  function animate() {
    requestAnimationFrame(animate)
    donut.rotation.x += 0.01
    donut.rotation.y += 0.01
    uniforms.time.value += 0.05
    controls.update()
    renderer.render(scene, camera)
  }

  animate()

  function onWindowResize() {
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
  }

  window.addEventListener('resize', onWindowResize)

  onUnmounted(() => {
    window.removeEventListener('resize', onWindowResize)
  })
})
</script>

<template>
  <div class="h-screen w-full bg-gray-900 flex items-center justify-center">
    <canvas ref="canvas" />
  </div>
</template>

