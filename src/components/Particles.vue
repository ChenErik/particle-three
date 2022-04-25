<script setup lang="ts">
import * as THREE from 'three'
let scene: THREE.Scene| null = null
let particles: THREE.Points| null = null
interface Props {
  amountX?: number
  amountY?: number
  size?: string
  color?: string
  top?: number
}
interface Data {
  count: number
  mouseX: number
  windowHalfX: number
  camera: THREE.PerspectiveCamera | null
  renderer: THREE.WebGLRenderer| null
}
const props = withDefaults(defineProps<Props>(), {
  amountX: 70,
  amountY: 70,
  size: '600.0',
  color: '#097bdb',
  top: 350,
})
const data: Data = {
  count: 0,
  mouseX: 0,
  windowHalfX: 0,
  camera: null,
  renderer: null,
}
function init() {
  const SEPARATION = 100
  const SCREEN_WIDTH = window.innerWidth
  const SCREEN_HEIGHT = window.innerHeight
  const container = document.createElement('div') as HTMLDivElement
  data.windowHalfX = window.innerWidth / 2
  container.style.position = 'realtive'
  container.style.top = `${props.top}px`
  container.style.height = `${(SCREEN_HEIGHT - props.top)}px`
  document.getElementById('particles')?.appendChild(container)
  data.camera = new THREE.PerspectiveCamera(75, SCREEN_WIDTH / SCREEN_HEIGHT, 1, 10000)
  data.camera.position.z = 1000
  scene = new THREE.Scene()
  const numParticles = props.amountX * props.amountY
  const positions = new Float32Array(numParticles * 3)
  const scales = new Float32Array(numParticles)
  let i = 0
  let j = 0
  for (let ix = 0; ix < props.amountX; ix++) {
    for (let iy = 0; iy < props.amountY; iy++) {
      positions[i] = ix * SEPARATION - ((props.amountX * SEPARATION) / 2)
      positions[i + 1] = 0
      positions[i + 2] = iy * SEPARATION - ((props.amountY * SEPARATION) / 2)
      scales[j] = 1
      i += 3
      j++
    }
  }
  const geometry = new THREE.BufferGeometry()
  geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  geometry.setAttribute('scale', new THREE.BufferAttribute(scales, 1))
  const material = new THREE.ShaderMaterial({
    uniforms: {
      color: { value: new THREE.Color(props.color) },
    },
    vertexShader: `
          attribute float scale;
          void main() {
            vec4 mvPosition = modelViewMatrix * vec4( position, 2.0 );
            gl_PointSize = scale * ( ${props.size} / - mvPosition.z );
            gl_Position = projectionMatrix * mvPosition;
          }
        `,
    fragmentShader: `
          uniform vec3 color;
          void main() {
            if ( length( gl_PointCoord - vec2( 0.5, 0.5 ) ) > 0.475 ) discard;
            gl_FragColor = vec4( color, 1.0 );
          }
        `,
  })
  particles = new THREE.Points(geometry, material)
  scene.add(particles)
  data.renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
  data.renderer.setSize(container.clientWidth, container.clientHeight)
  data.renderer.setPixelRatio(window.devicePixelRatio)
  data.renderer.setClearAlpha(0)
  container.appendChild(data.renderer.domElement)
  window.addEventListener('resize', onWindowResize, { passive: false })
  document.addEventListener('mousemove', onDocumentMouseMove, { passive: false })
  document.addEventListener('touchstart', onDocumentTouchStart, { passive: false })
  document.addEventListener('touchmove', onDocumentTouchMove, { passive: false })
}
function render() {
  data.camera!.position.x += (data.mouseX - data.camera!.position.x) * 0.05
  data.camera!.position.y = 400
  data.camera!.lookAt(scene!.position)
  const positions = particles.geometry.attributes.position.array as number[]
  const scales = particles.geometry.attributes.scale.array as number[]
  // 计算粒子位置及大小
  let i = 0
  let j = 0
  for (let ix = 0; ix < props.amountX; ix++) {
    for (let iy = 0; iy < props.amountY; iy++) {
      positions[i + 1] = (Math.sin((ix + data.count) * 0.3) * 100) + (Math.sin((iy + data.count) * 0.5) * 100)
      scales[j] = (Math.sin((ix + data.count) * 0.3) + 1) * 8 + (Math.sin((iy + data.count) * 0.5) + 1) * 8
      i += 3
      j++
    }
  }
  // 重新渲染粒子
  particles.geometry.attributes.position.needsUpdate = true
  particles.geometry.attributes.scale.needsUpdate = true
  data.renderer!.render(scene!, data.camera!)
  data.count += 0.1
}
function animate(): void {
  requestAnimationFrame(animate)
  render()
}
function onDocumentMouseMove(event: MouseEvent): void {
  data.mouseX = event.clientX - data.windowHalfX
}
function onDocumentTouchStart(event: TouchEvent) {
  if (event.touches.length === 1)
    data.mouseX = event.touches[0].pageX - data.windowHalfX
}
function onDocumentTouchMove(event: TouchEvent) {
  if (event.touches.length === 1) {
    event.preventDefault()
    data.mouseX = event.touches[0].pageX - data.windowHalfX
  }
}
function onWindowResize() {
  data.windowHalfX = window.innerWidth / 2
  data.camera!.aspect = window.innerWidth / window.innerHeight
  data.camera!.updateProjectionMatrix()
  data.renderer!.setSize(window.innerWidth, window.innerHeight)
}
onMounted(() => {
  init()
  animate()
})
</script>

<template>
  <div id="particles" />
</template>

<style>
#particles {
  background: url(../../public/bg.png);
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  z-index: -1;
  width: 100%;
  overflow: hidden;
}
</style>
