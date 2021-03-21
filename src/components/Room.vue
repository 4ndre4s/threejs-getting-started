<template>
  <div ref="webgl"></div>
</template>

<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import THREEx from '@/plugins/threex.keyboardstate'

export default {
  name: 'Room',
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

    const backWall = this.generateWall(5, 20, 2, 1, Math.PI, Math.PI / 2, Math.PI)
    backWall.position.x = -5
    backWall.position.z = -2.5

    const windowWall = this.generateWall(5, 20, 3, 1, Math.PI / 2, Math.PI / 2, Math.PI / 2, '/img/glass.jpg', 0.3)
    windowWall.position.x = 5
    windowWall.position.z = -2.5

    const leftWall = this.generateWall(5, 10, 1, 1)
    leftWall.position.x = 0
    leftWall.position.y = 10
    leftWall.position.z = -2.5

    const rightWall = this.generateWall(5, 10, 1, 1)
    rightWall.position.x = 0
    rightWall.position.y = -10
    rightWall.position.z = -2.5

    floor.add(backWall)
    floor.add(windowWall)
    floor.add(leftWall)
    floor.add(rightWall)
    floor.add(cube)
    scene.add(floor)

    const light = this.generatePointLight(0xffffff, 1)
    light.position.y = 5
    light.position.x = 15
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
      this.handleKeyPresses(step, scene)

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
      const texture = THREE.ImageUtils.loadTexture('/img/parkett.jpg')
      texture.wrapS = THREE.RepeatWrapping
      texture.wrapT = THREE.RepeatWrapping
      texture.repeat.set(2, 1)
      texture.rotation = Math.PI / 2

      const geometry = new THREE.PlaneGeometry(width, depth)
      const material = new THREE.MeshLambertMaterial(
        {
          map: texture,
          side: THREE.DoubleSide
        }
      )
      const mesh = new THREE.Mesh(geometry, material)
      mesh.receiveShadow = true
      return mesh
    },
    generateWall (width, depth, repeatX, repeatY, rotateX = Math.PI / 2, rotateY = Math.PI, rotateZ = Math.PI / 2, textureUrl = '/img/wall.jpg', opacity = 1) {
      const texture = THREE.ImageUtils.loadTexture(textureUrl)
      texture.wrapS = THREE.RepeatWrapping
      texture.wrapT = THREE.RepeatWrapping
      texture.repeat.set(repeatX, repeatY)
      texture.rotation = Math.PI / 2

      const geometry = new THREE.PlaneGeometry(width, depth)
      const material = new THREE.MeshLambertMaterial(
        {
          map: texture,
          side: THREE.DoubleSide
        }
      )

      material.transparent = opacity !== 1
      material.opacity = opacity

      const mesh = new THREE.Mesh(geometry, material)
      mesh.receiveShadow = true
      mesh.rotation.x = rotateX
      mesh.rotation.z = rotateZ
      mesh.rotation.y = rotateY
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
    },
    handleKeyPresses (step, scene) {
      const cube = scene.getObjectByName('cube')

      if (this.keyboard.pressed('o')) {
        if (cube.position.x > -4.25) {
          cube.translateX(-step)
        }
      }
      if (this.keyboard.pressed('u')) {
        if (cube.position.x < 4.25) {
          cube.translateX(step)
        }
      }
      if (this.keyboard.pressed('a')) {
        if (cube.position.y < 9.25) {
          cube.translateY(step)
        }
      }
      if (this.keyboard.pressed('e')) {
        if (cube.position.y > -9.25) {
          cube.translateY(-step)
        }
      }
    }
  }
}
</script>
