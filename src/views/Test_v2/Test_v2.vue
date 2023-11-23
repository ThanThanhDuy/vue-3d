<script setup lang="ts">
import * as THREE from "three";
import { ref, onMounted } from "vue";
import axios from "axios";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
const data = ref<any>({});
const colors = [
  "#ff0000", //
  "#00ff00", //
  "#ffc815", //
  "#433e90", //
  "#0f8880", //
  "#000000", //
  "#556f55", //
  "#38f29f", //
  "#ffaaff", //
  "#cafa1a", //
];
onMounted(async () => {
  const el = document.getElementById("three");
  const width = window.innerWidth,
    height = window.innerHeight;
  const scene = new THREE.Scene();
  const objectContainer = new THREE.Object3D();

  scene.background = new THREE.Color(0x000000);
  const renderer = new THREE.WebGLRenderer();
  renderer.setSize(width, height);
  document.getElementById("three").appendChild(renderer.domElement);

  // let res = await axios.get(
  //   "http://192.168.1.57:8767/analysis/data?id=a0923c22-7409-44bb-9568-23ca146e837e"
  // );
  let res = await axios.get(
    "http://192.168.1.57:8767/analysis/data?id=6dc7533b-90eb-458a-81a5-ce329b40bb1c"
  );
  data.value = res.data.data;
  let listMesh = [];
  for (let index = 0; index < data.value.faces.length; index++) {
    const face = data.value.faces[index]; // soft white light
    const geometry = new THREE.BufferGeometry();
    const vertices = new Float32Array(face.vertex);
    const normals = new Float32Array(face.normal);
    const indices = new Uint16Array(face.triangle);
    geometry.setAttribute("position", new THREE.BufferAttribute(vertices, 3));
    geometry.setAttribute("normal", new THREE.BufferAttribute(normals, 3));
    geometry.setIndex(new THREE.BufferAttribute(indices, 1));
    geometry.scale(0.1, 0.1, 0.1);
    // geometry.computeBoundingSphere();
    let material = new THREE.MeshBasicMaterial({
      color: colors[index],
      // wireframe: true,
    });
    listMesh[index] = new THREE.Mesh(geometry, material);
    const { x, y, z } = data.value.centroid;
    listMesh[index].position.x = x > 0 ? x / -10 : x / 10;
    listMesh[index].position.y = y > 0 ? y / 10 : y / -10;
    listMesh[index].position.z = z > 0 ? z / -10 : z / 10;
    const light = new THREE.AmbientLight(0x404040);
    scene.add(light);
    scene.add(listMesh[index]);
  }

  var initialZoom = 50;
  const camera = new THREE.OrthographicCamera(
    window.innerWidth / -initialZoom,
    window.innerWidth / initialZoom,
    window.innerHeight / initialZoom,
    window.innerHeight / -initialZoom,
    1,
    1000
  );
  camera.position.z = 1;

  let controls = new OrbitControls(camera, renderer.domElement);
  controls.dampingFactor = 50;
  controls.screenSpacePanning = false;
  camera.position.z = 5;
  let grid = new THREE.GridHelper(40, 40);
  scene.add(grid);
  const axesHelper = new THREE.AxesHelper(100);
  scene.add(axesHelper);
  function animation() {
    for (let j = 0; j < listMesh.length; j++) {
      // listMesh[j].rotation.x += 0.01;
      // listMesh[j].rotation.y += 0.01;
      // listMesh[j].position.y += 0.01;
    }

    renderer.render(scene, camera);
  }
  renderer.setAnimationLoop(animation);
});
</script>

<template>
  <div style="display: flex; height: 100%">
    <div style="background: #fff; flex: 1; height: 100vh" id="three"></div>
  </div>
</template>
