<script setup lang="ts">
import * as THREE from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import { onMounted, ref } from "vue";
import axios from "axios";
const data = ref<any>({});
onMounted(async () => {
  const ele = document.getElementById("three");
  if (ele) {
    console.log(ele?.offsetWidth, ele?.offsetHeight);
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0xffffff);
    const camera = new THREE.PerspectiveCamera(
      40,
      ele?.offsetWidth / ele?.offsetHeight,
      0.1,
      1000
    );

    const renderer = new THREE.WebGLRenderer();
    renderer.shadowMap.enabled = true;
    renderer.setSize(ele?.offsetWidth, ele?.offsetHeight);
    ele.appendChild(renderer.domElement);

    let res = await axios.get(
      "http://192.168.1.57:8767/analysis/data?id=6dc7533b-90eb-458a-81a5-ce329b40bb1c"
    );
    let listMesh: any = [];
    // const colors = [
    //   "#ff0000", //
    //   "#00ff00", //
    //   "#ffc815", //
    //   "#433e90", //
    //   "#0f8880", //
    //   "#000000", //
    //   "#556f55", //
    //   "#38f29f", //
    //   "#ffaaff", //
    //   "#cafa1a", //
    // ];
    data.value = res.data.data;
    const totalBoundingBox = new THREE.Box3();
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
      let material = new THREE.MeshBasicMaterial({ color: "#adb2bd" });
      listMesh[index] = new THREE.Mesh(geometry, material);

      var geometry_line = new THREE.EdgesGeometry(listMesh[index].geometry);
      var material_line = new THREE.LineBasicMaterial({
        color: "#919191",
        linewidth: 2,
      });
      let edges = new THREE.LineSegments(geometry_line, material_line);
      listMesh[index].add(edges);
      const { x, y, z } = data.value.centroid;
      listMesh[index].position.x = x > 0 ? x / -10 : x / 10;
      listMesh[index].position.y = y > 0 ? y / 10 : y / -10;
      listMesh[index].position.z = z > 0 ? z / -10 : z / 10;
      listMesh[index].castShadow = true;
      // camera

      const cubeBoundingBox = new THREE.Box3().setFromObject(listMesh[index]);
      if (
        cubeBoundingBox.max.x > 0 &&
        cubeBoundingBox.max.y > 0 &&
        cubeBoundingBox.max.z > 0
      ) {
        totalBoundingBox.expandByPoint(cubeBoundingBox.min);
        totalBoundingBox.expandByPoint(cubeBoundingBox.max);
      }

      scene.add(listMesh[index]);
    }
    // const { x, y, z } = data.value.boundBox;
    console.log(totalBoundingBox);
    const maxObjectSize = Math.max(
      totalBoundingBox.max.x,
      totalBoundingBox.max.y,
      totalBoundingBox.max.z
    );
    console.log(maxObjectSize);

    // Tính toán khoảng cách cần di chuyển camera để vật thể nằm trong tầm nhìn
    const distance =
      (maxObjectSize * 2) / Math.tan(THREE.MathUtils.degToRad(camera.fov) / 2);
    console.log(distance);

    // Đặt vị trí và hướng nhìn mới cho camera
    camera.position.set(distance, distance, distance);
    var raycaster = new THREE.Raycaster();
    var mouse = new THREE.Vector2();

    function onDocumentMouseDown(event: MouseEvent) {
      event.preventDefault();
      console.log(camera.position);
      const uuids: string[] = [];
      scene.children.forEach((e: any) => uuids.push(e.uuid));
      mouse.x = (event.clientX / renderer.domElement.clientWidth) * 2 - 1;
      mouse.y = -(event.clientY / renderer.domElement.clientHeight) * 2 + 1;

      raycaster.setFromCamera(mouse, camera);
      let intersects = raycaster.intersectObjects(scene.children);
      if (intersects.length > 0) {
        scene.children.forEach((item: any) =>
          item.material.color.set("#adb2bd")
        );
        for (let i = 0; i < intersects.length; i++) {
          const el = intersects[i];
          if (uuids.findIndex(e => e == el.object.uuid) != -1) {
            el.object.material.color.set("#8bc4ea");
            return;
          }
        }
      }
    }
    window.addEventListener("click", onDocumentMouseDown, false);
    // // grid
    let grid = new THREE.GridHelper(20, 20);
    scene.add(grid);
    // control object
    let controls = new OrbitControls(camera, renderer.domElement);
    controls.dampingFactor = 50;
    controls.screenSpacePanning = false;
    // axesHelper
    // const axesHelper = new THREE.AxesHelper(100);
    // scene.add(axesHelper);
    // plane
    // const planeGeometry = new THREE.PlaneGeometry(40, 40);
    // const planeMaterial = new THREE.MeshBasicMaterial({ color: 0xffffff });
    // const plane = new THREE.Mesh(planeGeometry, planeMaterial);
    // scene.add(plane);
    // plane.rotation.x = -0.5 * Math.PI;
    // plane.receiveShadow = true;

    //camera.position.z = 3;
    function animate() {
      requestAnimationFrame(animate);
      renderer.render(scene, camera);
    }

    animate();
  }
});
</script>

<template>
  <div style="display: flex; padding-top: 32px">
    <div style="width: 100%; height: calc(100vh - 32px)" id="three"></div>
    <!-- <div style="width: 50%; height: 100vh; background-color: beige"></div> -->
  </div>
</template>
