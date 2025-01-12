<template>
  <div class="donut-container">
    <Renderer ref="renderer" resize="window" antialias :orbit-ctrl="{ enableDamping: true, dampingFactor: 0.05 }">
      <Camera :position="{ z: 5 }" />
      <Scene background="#111111">
        <PointLight :position="{ x: 10, y: 10, z: 10 }" :intensity="1.5" :cast-shadow="true" />
        <AmbientLight :intensity="0.2" />
        <Torus ref="torus" :radius="1" :tube="0.4" :radial-segments="32" :tubular-segments="100" :rotation="{ x: 0, y: 0, z: 0 }">
          <ShaderMaterial :vertex-shader="vertexShader" :fragment-shader="fragmentShader" :uniforms="uniforms" />
        </Torus>
        <Plane :width="10" :height="10" :rotation="{ x: -Math.PI / 2, y: 0, z: 0 }" :position="{ y: -2 }" receive-shadow>
          <BasicMaterial transparent :opacity="0.5" />
        </Plane>
      </Scene>
    </Renderer>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, onUnmounted } from 'vue'
import { Renderer, Camera, Scene, Torus, PointLight, AmbientLight, Plane, ShaderMaterial, BasicMaterial } from 'troisjs'
import * as THREE from 'three'

export default defineComponent({
  name: 'DonutRing',
  components: {
    Renderer,
    Camera,
    Scene,
    Torus,
    PointLight,
    AmbientLight,
    Plane,
    ShaderMaterial,
    BasicMaterial
  },
  setup() {
    const renderer = ref(null)
    const torus = ref(null)

    const uniforms = {
      time: { value: 0 },
      color1: { value: new THREE.Color('#69ff9d') },
      color2: { value: new THREE.Color('#2ecc71') }
    }

    const vertexShader = `
      varying vec2 vUv;
      void main() {
        vUv = uv;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
      }
    `

    const fragmentShader = `
      uniform float time;
      uniform vec3 color1;
      uniform vec3 color2;
      varying vec2 vUv;
      void main() {
        vec3 color = mix(color1, color2, vUv.x + sin(time) * 0.3);
        gl_FragColor = vec4(color, 1.0);
      }
    `

    let animationFrameId: number

    const animate = () => {
      if (torus.value) {
        torus.value.rotation.x += 0.01
        torus.value.rotation.y += 0.01
      }
      uniforms.time.value += 0.05
      animationFrameId = requestAnimationFrame(animate)
      renderer.value?.render()
    }

    onMounted(() => {
      animate()
    })

    onUnmounted(() => {
      cancelAnimationFrame(animationFrameId)
    })

    return {
      renderer,
      torus,
      uniforms,
      vertexShader,
      fragmentShader
    }
  }
})
</script>

<style scoped>
.donut-container {
  width: 100%;
  height: 100vh;
  background-color: #111111;
}
</style>
