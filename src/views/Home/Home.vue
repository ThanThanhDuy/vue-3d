<script setup lang="ts">
import * as THREE from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import { onMounted, ref } from "vue";
import axios from "axios";
const data = ref<any>({});
onMounted(async () => {
  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0xffffff);
  const camera = new THREE.PerspectiveCamera(
    45,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );

  const renderer = new THREE.WebGLRenderer();
  renderer.shadowMap.enabled = true;
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  let res = await axios.get(
    "http://192.168.1.57:8767/analysis/data?id=b6ed3d7a-5e0f-41d2-99e8-51728ad55966"
  );
  let listMesh: any = [];
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
  data.value = res.data.data;
  for (let index = 0; index < data.value.faces.length; index++) {
    const face = data.value.faces[index];
    const geometry = new THREE.BufferGeometry();
    const vertices = new Float32Array(face.vertex);
    const normals = new Float32Array(face.normal);
    const indices = new Uint16Array(face.triangle);
    geometry.setAttribute("position", new THREE.BufferAttribute(vertices, 3));
    geometry.setAttribute("normal", new THREE.BufferAttribute(normals, 3));
    geometry.setIndex(new THREE.BufferAttribute(indices, 1));
    geometry.scale(0.1, 0.1, 0.1);
    let material = new THREE.MeshBasicMaterial({ color: "#9ca3af" });

    const edges = new THREE.EdgesGeometry(geometry);
    const line = new THREE.LineSegments(
      edges,
      new THREE.LineBasicMaterial({ color: "#6b7280" })
    );
    scene.add(line);
    listMesh[index] = new THREE.Mesh(geometry, material);

    const { x, y, z } = data.value.centroid;
    line.position.x = x > 0 ? x / -10 : x / 10;
    line.position.y = y > 0 ? y / 10 : y / -10;
    line.position.z = z > 0 ? z / -10 : z / 10;
    listMesh[index].position.x = x > 0 ? x / -10 : x / 10;
    listMesh[index].position.y = y > 0 ? y / 10 : y / -10;
    listMesh[index].position.z = z > 0 ? z / -10 : z / 10;
    listMesh[index].castShadow = true;
    scene.add(listMesh[index]);
  }
  // // grid
  // let grid = new THREE.GridHelper(40, 40);
  // scene.add(grid);
  // control object
  let controls = new OrbitControls(camera, renderer.domElement);
  controls.dampingFactor = 50;
  controls.screenSpacePanning = false;
  // axesHelper
  const axesHelper = new THREE.AxesHelper(100);
  scene.add(axesHelper);
  // plane
  // const planeGeometry = new THREE.PlaneGeometry(40, 40);
  // const planeMaterial = new THREE.MeshBasicMaterial({ color: 0xffffff });
  // const plane = new THREE.Mesh(planeGeometry, planeMaterial);
  // scene.add(plane);
  // plane.rotation.x = -0.5 * Math.PI;
  // plane.receiveShadow = true;

  camera.position.z = 50;
  function animate() {
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
  }

  animate();
});
</script>

<template>
  <div></div>
</template>
