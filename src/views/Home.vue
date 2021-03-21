<template>
  <div ref="webgl"></div>
</template>

<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import THREEx from '@/plugins/threex.keyboardstate'

export default {
  name: 'Home',
  data () {
    return {
      keyboard: new THREEx.KeyboardState(),
      clock: new THREE.Clock()
    }
  },
  mounted () {
    const scene = new THREE.Scene()

    const cube = this.generateCube(1, 1, 1)
    cube.name = 'cube'
    cube.translateZ(-2)
    cube.position.y = cube.geometry.parameters.height / 2

    const floor = this.generateFloor(10, 20)
    floor.name = 'floor'
    floor.rotation.x = Math.PI / 2

    floor.add(cube)
    scene.add(floor)

    const light = this.generatePointLight(0xffffff, 1)
    light.position.y = 5
    light.position.x = 5
    scene.add(light)

    const camera = this.setupCamera()

    const renderer = this.setupRenderer()
    this.$refs.webgl.appendChild(renderer.domElement)
    const controls = new OrbitControls(camera, renderer.domElement)
    this.update(renderer, scene, camera, controls)
  },
  methods: {
    update (renderer, scene, camera, controls) {
      renderer.render(scene, camera)
      controls.update()

      const step = 10 * this.clock.getDelta()
      const cube = scene.getObjectByName('cube')
      if (this.keyboard.pressed('o')) {
        cube.translateX(-step)
      }
      if (this.keyboard.pressed('u')) {
        cube.translateX(step)
      }
      if (this.keyboard.pressed('a')) {
        cube.translateY(step)
      }
      if (this.keyboard.pressed('e')) {
        cube.translateY(-step)
      }

      requestAnimationFrame(() => {
        this.update(renderer, scene, camera, controls)
      })
    },
    generatePointLight (color, intensity) {
      const light = new THREE.PointLight(color, intensity)
      light.castShadow = true
      return light
    },
    setupCamera () {
      const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 1000)
      camera.position.x = 10
      camera.position.y = 15
      camera.position.z = 15
      camera.lookAt(new THREE.Vector3(0, 0, -5))
      return camera
    },
    generateFloor (width, depth) {
      const geometry = new THREE.PlaneGeometry(width, depth)
      const material = new THREE.MeshPhongMaterial(
        {
          color: 'rgb(100,100,100)',
          side: THREE.DoubleSide
        }
      )
      const mesh = new THREE.Mesh(geometry, material)
      mesh.receiveShadow = true
      return mesh
    },
    generateCube (width, height, depth) {
      const geometry = new THREE.BoxGeometry(width, height, depth)
      const material = new THREE.MeshPhongMaterial({ color: 'rgb(200, 100, 100)' })
      const mesh = new THREE.Mesh(geometry, material)
      mesh.castShadow = true
      return mesh
    },
    setupRenderer () {
      const renderer = new THREE.WebGLRenderer()
      renderer.shadowMap.enabled = true
      renderer.setSize(window.innerWidth, window.innerHeight)
      renderer.setClearColor('rgb(80,80,80)')
      return renderer
    }
  }
}
</script>
