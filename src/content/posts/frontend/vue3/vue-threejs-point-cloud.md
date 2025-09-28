---
title: Vue+Threejs实现点云效果
published: 2025-06-11
image: https://images.unsplash.com/photo-1749254080180-f9da0512b5c7?ixid=M3wzNTk3NzR8MHwxfHRvcGljfHxibzhqUUtUYUUwWXx8fHx8Mnx8MTc0OTYzMTAzNHw&ixlib=rb-4.1.0
tags: [ "Vue3", "可视化","Threejs" ]
category: 前端
---

# index.vue

```vue
<script setup lang="ts">
  import { useWave } from '.'

  const { init } = useWave()

  onMounted(() => {
    init(document.querySelector('#wave-container'))
  })
</script>

<template>
  <div id="wave-container" class="h-1/3 w-full"></div>
</template>

<style lang="scss" scoped></style>
```

# index.ts

```typescript
import * as THREE from 'three'

export function useWave() {
  const scene: THREE.Scene = new THREE.Scene()
  const renderer: THREE.WebGLRenderer = new THREE.WebGLRenderer({
    antialias: true,
    alpha: true,
  })
  let camera: THREE.PerspectiveCamera

  const GAP = 100
  const AMOUNT_X = 50
  const AMOUNT_Y = 50

  let count = 0
  let particles: THREE.Points

  function init(container: HTMLDivElement | null) {
    // 如果container为空，抛出错误
    if (!container) {
      throw new Error('no container')
    }

    // 创建一个透视相机，参数为视角、宽高比、近裁剪面、远裁剪面
    camera = new THREE.PerspectiveCamera(
      60,
      container.clientWidth / container.clientHeight,
      0.1,
      10000,
    )
    // 设置渲染器的大小
    renderer.setSize(container.clientWidth, container.clientHeight)
    // 将渲染器的dom元素添加到container中
    container.appendChild(renderer.domElement)

    // 设置相机的位置
    camera.position.z = 1000
    camera.position.y = 500
    camera.scale.set(0.5, 1, 1)

    // 计算粒子数量
    const particlesCount = AMOUNT_X * AMOUNT_Y

    const positions = new Float32Array(particlesCount * 3)
    const scales = new Float32Array(particlesCount)

    let i = 0
    let j = 0

    // 循环计算每个粒子的位置和大小
    for (let ix = 0; ix < AMOUNT_X; ix++) {
      for (let iy = 0; iy < AMOUNT_Y; iy++) {
        positions[i] = ix * GAP - (AMOUNT_X * GAP) / 2 // x
        positions[i + 1] = 0 // y
        positions[i + 2] = iy * GAP - (AMOUNT_Y * GAP) / 2 // z

        scales[j] = 1

        i += 3
        j++
      }
    }

    // 创建一个缓冲几何体，并设置位置和大小属性
    const geometry = new THREE.BufferGeometry()
    geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
    geometry.setAttribute('scale', new THREE.BufferAttribute(scales, 1))

    // 创建一个着色器材质，并设置顶点着色器和片元着色器
    const material = new THREE.ShaderMaterial({
      uniforms: {
        color: { value: new THREE.Color(0x4096ff) },
      },
      vertexShader: `
        attribute float scale;

        void main() {
          vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );
          gl_PointSize = scale * ( 300.0 / - mvPosition.z );
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

    // 创建一个点云，并添加到场景中
    particles = new THREE.Points(geometry, material)
    scene.add(particles)

    // 设置渲染器的大小和像素比例，并设置动画循环
    renderer.setSize(container.clientWidth, container.clientHeight)
    renderer.setPixelRatio(window.devicePixelRatio)
    renderer.setAnimationLoop(render)

    // 将渲染器的dom元素添加到container中，并添加窗口大小改变事件监听器
    container.appendChild(renderer.domElement)
    window.addEventListener('resize', () => onWindowResize(container))
  }

  // 渲染函数
  function render() {
    // 将相机对准场景的位置
    camera.lookAt(scene.position)
    // 获取粒子的位置和缩放属性数组
    const positions = particles.geometry.attributes.position.array
    const scales = particles.geometry.attributes.scale.array
    // 初始化索引
    let i = 0
    let j = 0
    // 遍历粒子的位置和缩放属性数组
    for (let ix = 0; ix < AMOUNT_X; ix++) {
      for (let iy = 0; iy < AMOUNT_Y; iy++) {
        // 计算粒子的位置
        positions[i + 1] =
          Math.sin((ix + count) * 0.3) * 120 +
          Math.sin((iy + count) * 0.5) * 120
        // 计算粒子的缩放
        scales[j] =
          (Math.sin((ix + count) * 0.3) + 1) * 5 +
          (Math.sin((iy + count) * 0.5) + 1) * 5
        // 更新索引
        i += 3
        j++
      }
    }
    // 通知位置和缩放属性数组已经更新
    particles.geometry.attributes.position.needsUpdate = true
    particles.geometry.attributes.scale.needsUpdate = true
    // 渲染场景
    renderer.render(scene, camera)
    // 更新计数器
    count += 0.05
  }

  function onWindowResize(container: HTMLDivElement) {
    camera.aspect = container.clientWidth / container.clientHeight
    camera.updateProjectionMatrix()
    renderer.setSize(container.clientWidth, container.clientHeight)
  }

  return {
    init,
  }
}
```
