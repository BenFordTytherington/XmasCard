
<script>

import { 
  Renderer, 
  Camera, 
  Scene,
  Cylinder,
  LambertMaterial,
  BasicMaterial,
  PointLight,
  AmbientLight,
  GltfModel,
  Texture,
  SphereGeometry,
  Mesh,
  Group,
  RenderPass,
  UnrealBloomPass,
  EffectComposer,
  BoxGeometry,
  InstancedMesh,
  DirectionalLight, 
  Box,
} from 'troisjs'
import { Color, Object3D } from 'three'





export default {
  mounted() {
    const renderer = this.$refs.renderer
    const orbitCtrl = renderer.three.cameraCtrl
    orbitCtrl.autoRotate = true  

    const lights = []

    for (let i = 1; i <= 120; i++) {
      let refname = 'light' + i
      lights.push(
        this.$refs[refname][0].light
      )
    }

    lights.forEach(l => {
      let colour = this.getColour()
      let mat = l.children[0].material
      mat.color = l.color = colour
      l.position.y += 1
    })

    const snowMesh = this.$refs.snow.mesh
    const inst = new Object3D()

    let t = 0
    let positions = []
    let rates = []

    for (let i = 0; i < 60; i++) {
      this.treePositions[i] = {x: Math.random() * 80 - 40, y: 4, z: Math.random() * 80 - 40}
    }

    for (let i = 0; i < 160; i++) {
      let index = Math.floor(Math.random() * 30)
      let pos = this.treePositions[index]

      this.lightPositions[i] = {
        x: pos.x + Math.random() * 6 - 3,
        y: pos.y + Math.random() * 2 + 2,
        z: pos.z + Math.random() * 6 - 3}
    }


    for (let i = 0; i < 500; i++) {
      positions[i] = {
        x: Math.random() * 80 - 45,
        y: 60,
        z: Math.random() * 80 - 45
      }
      rates[i] = Math.random() * 0.5
    }
    
    renderer.onBeforeRender(() => {

      for (let i = 0; i < 500; i++) {
        if (positions[i].y <= 8) {
          positions[i].y = 50 + Math.random() * 10
          positions[i].x = Math.random() * 80 - 45
          positions[i].z = Math.random() * 80 - 45
          rates[i] = Math.random() * 0.5
        }

        positions[i].y -= 0.75 * rates[i]   
        positions[i].x += Math.cos(t + (Math.random() * 8)) * Math.random()
        positions.z += Math.sin(t * 0.1) * 0.125
        rates[i] += 0.008

        inst.position.x = positions[i].x
        inst.position.y = positions[i].y
        inst.position.z = positions[i].z

        inst.updateMatrix()
        snowMesh.setMatrixAt(i, inst.matrix)
      }
      snowMesh.instanceMatrix.needsUpdate = true
      
      t += 0.1

    })



  },
  data() {
    return {
      lastCol: 0x000000,
      lastPos: {x: 0, y: 0, z: 0},
      treePositions: [],
      lightPositions: []
    }
  },

  components: {
    Renderer,
    Camera,
    Scene,
    Cylinder,
    LambertMaterial,
    BasicMaterial,
    PointLight,
    AmbientLight,
    GltfModel,
    Texture,
    SphereGeometry,
    Mesh,
    Group,
    RenderPass,
    UnrealBloomPass,
    EffectComposer,
    BoxGeometry,
    InstancedMesh,
    DirectionalLight,
    Box,
},
  methods: {
    getScale(value) {
      return {x: value, y: value, z: value}
    },

    getPosCircle(t, x, y, z) {
      return {
        x: x + 1.5 * Math.sin(t),
        y: y + 3,
        z: z + 1.5 * Math.cos(t) }
    },

    getColour() {
      let colours = [0xBF0F0F, 0x0FBF0F, 0x0F0FBF, 0xAF0F0F, 0x0AFF0F, 0x0AFFFF, 0xFAFAFA]

      let index = Math.floor((Math.random() * 7))

      let pCol = colours[index]

      return new Color(pCol)

    }
  }
}
</script>

<template>
  <Renderer ref='renderer' :orbitCtrl="true" resize="window">
    <Camera ref='cam' :position="{x: 0, y: 65, z: 95}" :lookAt="this.$refs.ground"/>
    <Scene :background="0x232226">

      <AmbientLight :intensity="0.35" :color="0xFFFAFA" />

      <InstancedMesh ref="snow" :count="500">
        <BoxGeometry  :size="0.1"/>
        <BasicMaterial />
      </InstancedMesh>

      <DirectionalLight :position="{x: 3, y: 2, z: 2}" :intensity="0.75" :color="0xAAAAFF" />

      <GltfModel src="Christmas Tree.glb" v-for="i in 60" :scale="getScale(0.2 + (Math.random() * 0.1))" 
      :position="this.treePositions[i - 1]" />
      
      <PointLight v-for="i in 160" :ref="`light${i}`" :intensity="0.85" :position="this.lightPositions[i - 1]" :scale="getScale(0.15 + (0.2 * Math.random()))" :distance="9">
        <Mesh>
          <SphereGeometry />
          <BasicMaterial />
        </Mesh>
      </PointLight>

      
      <Cylinder ref='ground' :radiusTop="50" :radiusBottom="55" :height="8" :radialSegments="64" :heightSegments="32">
        <LambertMaterial >
          <Texture src="woodTex.jpg" />
        </LambertMaterial>
      </Cylinder>

      <Box :position="{x: 0, y: 35, z: 0}" :scale="{x: 20, y: 20, z: 20}">
        <LambertMaterial>
          <Texture src="tex.jpg"/>
        </LambertMaterial>
      </Box>

    </Scene>
    <EffectComposer>
      <RenderPass />
      <UnrealBloomPass :strength="0.5" :threshold="0.35" />
    </EffectComposer>
  </Renderer>
</template>