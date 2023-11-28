<script setup lang="ts">
import * as THREE from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
import { onMounted, ref } from "vue";
import axios from "axios";
const data = ref<any>({});
onMounted(async () => {
  const ele = document.getElementById("three");
  const ele_left = document.getElementById("left");
  const ele_right = document.getElementById("right");
  if (ele) {
    const scene = new THREE.Scene();
    // set background scene
    scene.background = new THREE.Color(0xffffff);
    // setup default camera
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

    data.value = res.data.data;
    data.value.faces = data.value.faces.map(it => {
      return {
        ...it,
        isShow: true,
      };
    });
    const totalBoundingBox = new THREE.Box3();
    const cameraBoundingBox = new THREE.Box3();
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
      let material = new THREE.MeshPhongMaterial({ color: "#ffffff" });
      listMesh[index] = new THREE.Mesh(geometry, material);
      // var geometry_line = new THREE.EdgesGeometry(listMesh[index].geometry);
      // var material_line = new THREE.LineBasicMaterial({
      //   color: "#919191",
      //   linewidth: 2,
      // });
      // let edges = new THREE.LineSegments(geometry_line, material_line);
      // listMesh[index].add(edges);

      // const { x, y, z } = data.value.centroid;
      // listMesh[index].position.x = x > 0 ? x / -10 : x / 10;
      // listMesh[index].position.y = y > 0 ? y / 10 : y / -10;
      // listMesh[index].position.z = z > 0 ? z / -10 : z / 10;
      listMesh[index].castShadow = true;
      const cubeBoundingBox = new THREE.Box3().setFromObject(listMesh[index]);
      if (
        isFinite(cubeBoundingBox.max.x) &&
        isFinite(cubeBoundingBox.max.y) &&
        isFinite(cubeBoundingBox.max.z)
      ) {
        totalBoundingBox.expandByPoint(cubeBoundingBox.min);
        totalBoundingBox.expandByPoint(cubeBoundingBox.max);
      }

      scene.add(listMesh[index]);
    }
    for (let i = 0; i < listMesh.length; i++) {
      listMesh[i].position.x =
        (totalBoundingBox.max.x + totalBoundingBox.min.x) / -2;
      listMesh[i].position.y =
        Math.abs(totalBoundingBox.max.y + totalBoundingBox.min.y) / 2;
      listMesh[i].position.z =
        (totalBoundingBox.max.z + totalBoundingBox.min.z) / -2;
      // const cubeHelper = new THREE.BoxHelper(listMesh[i], 0xffff00);
      // cubeHelper.position.x =
      //   (totalBoundingBox.max.x + totalBoundingBox.min.x) / -2;
      // cubeHelper.position.y =
      //   Math.abs(totalBoundingBox.max.y + totalBoundingBox.min.y) / 2;
      // cubeHelper.position.z =
      //   (totalBoundingBox.max.z + totalBoundingBox.min.z) / -2;
      // scene.add(cubeHelper);
    }
    for (let i = 0; i < listMesh.length; i++) {
      const cubeBoundingBox = new THREE.Box3().setFromObject(listMesh[i]);
      if (cubeBoundingBox.max.x > 0) {
        cameraBoundingBox.expandByPoint(cubeBoundingBox.min);
        cameraBoundingBox.expandByPoint(cubeBoundingBox.max);
      }
    }
    const maxObjectSize = Math.max(
      cameraBoundingBox.max.x,
      cameraBoundingBox.max.y,
      cameraBoundingBox.max.z
    );

    const distance =
      (maxObjectSize * 2) / Math.tan(THREE.MathUtils.degToRad(camera.fov) / 2);
    // console.log("distance", distance);

    // Đặt vị trí và hướng nhìn mới cho camera
    camera.position.set(distance, distance, distance);
    // nguồn sáng
    const directionalLight = new THREE.DirectionalLight(0xffffff, 2);
    directionalLight.position.set(3, 3, 10); // Đặt vị trí của nguồn sáng
    var lightHolder = new THREE.Group();
    lightHolder.add(directionalLight);
    scene.add(lightHolder);

    var raycaster = new THREE.Raycaster();
    var mouse = new THREE.Vector2();

    function onDocumentMouseDown(event: MouseEvent) {
      event.preventDefault();
      // console.log(camera.position);
      const uuids: string[] = [];
      scene.children.forEach((e: any) => uuids.push(e.uuid));
      console.log(ele_left?.offsetWidth);

      mouse.x =
        (event.clientX /
          (renderer.domElement.clientWidth +
            ele_left?.offsetWidth +
            ele_right?.offsetWidth)) *
          2 -
        1;

      mouse.y = -(event.clientY / renderer.domElement.clientHeight) * 2 + 1;
      raycaster.setFromCamera(mouse, camera);
      let intersects = raycaster.intersectObjects(scene.children);
      if (intersects.length > 0) {
        for (let i = 0; i < intersects.length; i++) {
          const el = intersects[i];
          if (uuids.findIndex(e => e == el.object.uuid) != -1) {
            if (el.object.type !== "BoxHelper") {
              scene.children.forEach((item: any) => {
                if (item.type === "Mesh") {
                  item.material.color.set("#ffffff");
                }
              });
              // console.log(el);
              el.object.material.color.set("#40ed6e");
            }
            return;
          }
        }
      }
    }
    window.addEventListener("click", onDocumentMouseDown, false);
    // // grid
    // let grid = new THREE.GridHelper(20, 20);
    // scene.add(grid);
    // control object
    let controls = new OrbitControls(camera, renderer.domElement);
    controls.dampingFactor = 50;
    controls.screenSpacePanning = false;
    function animate() {
      requestAnimationFrame(animate);
      lightHolder.quaternion.copy(camera.quaternion);
      renderer.render(scene, camera);
    }
    animate();
  }
});
</script>

<template>
  <div style="display: flex">
    <div
      style="width: 15%; height: 100vh; background-color: #ccc; z-index: 1"
      id="left"
    >
      <div v-for="item in data.faces" :key="item.id" style="padding: 15px">
        <div>
          <span style="margin-right: 15px">{{ item.index + 1 }}</span>
          <span style="margin-right: 15px">{{
            item.isShow ? "Show" : "Hidden"
          }}</span>
          <button>Change</button>
        </div>
      </div>
    </div>
    <div style="width: 85%; height: 100vh" id="three"></div>
    <div
      style="width: 15%; height: 100vh; background-color: #ccc; z-index: 1"
      id="right"
    ></div>
    <!-- <div style="width: 50%; height: 100vh; background-color: beige"></div> -->
  </div>
</template>
