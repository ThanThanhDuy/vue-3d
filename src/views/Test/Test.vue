<script setup lang="ts">
import * as THREE from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import { onMounted, ref } from "vue";
import axios from "axios";
const data = ref<any>({});
onMounted(async () => {
  // Tạo scene, camera, và renderer
  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0xffffff);
  const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );
  const renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);
  let res = await axios.get(
    "http://192.168.1.57:8767/analysis/data?id=b6ed3d7a-5e0f-41d2-99e8-51728ad55966"
  );
  data.value = res.data.data;
  let listMesh: any = [];
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
    let material = new THREE.MeshPhongMaterial({ color: 0x00ff00 });
    listMesh[index] = new THREE.Mesh(geometry, material);
    //   var geometry_line = new THREE.EdgesGeometry(listMesh[index].geometry);
    //   var material_line = new THREE.LineBasicMaterial({
    //     color: "#919191",
    //     linewidth: 2,
    //   });
    //   let edges = new THREE.LineSegments(geometry_line, material_line);
    //   listMesh[index].add(edges);
    const { x, y, z } = data.value.centroid;
    listMesh[index].position.x = x > 0 ? x / -10 : x / 10;
    listMesh[index].position.y = y > 0 ? y / 10 : y / -10;
    listMesh[index].position.z = z > 0 ? z / -10 : z / 10;
    scene.add(listMesh[index]);
  }

  // const geometry = new THREE.BoxGeometry();
  // const material = new THREE.MeshPhongMaterial({ color: 0x00ff00 });
  // const cube = new THREE.Mesh(geometry, material);
  // scene.add(cube);

  // Tạo directional light để chiếu sáng vật thể
  const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
  directionalLight.position.set(3, 3, 10); // Đặt vị trí của nguồn sáng
  var lightHolder = new THREE.Group();
  lightHolder.add(directionalLight);
  scene.add(lightHolder);

  // Tạo directional light helper
  const lightHelper = new THREE.DirectionalLightHelper(directionalLight, 5);
  scene.add(lightHelper);

  // Tạo OrbitControls
  const controls = new OrbitControls(camera, renderer.domElement);

  // Đặt màu nền xám

  // Đặt vị trí của camera
  camera.position.z = 5;

  // Hàm vòng lặp render
  const animate = function () {
    requestAnimationFrame(animate);
    lightHolder.quaternion.copy(camera.quaternion);
    // Xoay vật thể
    // cube.rotation.x += 0.01;
    // cube.rotation.y += 0.01;

    // Cập nhật controls để hỗ trợ quay và zoom
    controls.update();

    renderer.render(scene, camera);
  };

  // Gọi hàm animate
  animate();
});
</script>

<template></template>
