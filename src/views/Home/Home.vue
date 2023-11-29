<script setup lang="ts">
import * as THREE from "three";
import {OrbitControls} from "three/examples/jsm/controls/OrbitControls.js";
import {onMounted, ref} from "vue";

import axios from "axios";

const data = ref<any>({});
const selected = ref<string>('');
const renderModel3D = (data: any) => {
  data.value = data
  const ele = document.getElementById("three");
  ele!.innerHTML = ''
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

    let listMesh: any = [];

    const totalBoundingBox = new THREE.Box3();
    const cameraBoundingBox = new THREE.Box3();
    const group = new THREE.Group();
    for (let index = 0; index < data.faces.length; index++) {
      const face = data.faces[index];
      const geometry = new THREE.BufferGeometry();
      const vertices = new Float32Array(face.vertex);
      const normals = new Float32Array(face.normal);
      const indices = new Uint16Array(face.triangle);
      geometry.setAttribute("position", new THREE.BufferAttribute(vertices, 3));
      geometry.setAttribute("normal", new THREE.BufferAttribute(normals, 3));
      geometry.setIndex(new THREE.BufferAttribute(indices, 1));
      geometry.scale(0.1, 0.1, 0.1);
      let material = new THREE.MeshPhongMaterial({color: "#ffffff"});
      // if (!face.isShow) {
      material.transparent = true
      material.opacity = 0.1
      // }
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
      data.value.faces[index].uuid = listMesh[index].uuid
      // scene.add(listMesh[index]);
      group.add(listMesh[index])
    }
    scene.add(group)
    console.log('x', (totalBoundingBox.max.x + totalBoundingBox.min.x) / -2)
    console.log('y  ', Math.abs(totalBoundingBox.max.y + totalBoundingBox.min.y) / 2)
    console.log('z', (totalBoundingBox.max.z + totalBoundingBox.min.z) / -2)
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

    // caculate distance of camera
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
    // Đặt vị trí và hướng nhìn mới cho camera
    camera.position.set(distance, distance, distance);
    const box = new THREE.BoxHelper(group, 0x000000);
    box.position.x =
        (totalBoundingBox.max.x + totalBoundingBox.min.x) / -2;
    box.position.y =
        Math.abs(totalBoundingBox.max.y + totalBoundingBox.min.y) / 2;
    box.position.z =
        (totalBoundingBox.max.z + totalBoundingBox.min.z) / -2;

    // scene.add(box)
    let boundingBox = new THREE.Box3();
    boundingBox.setFromObject(box);
    const low = boundingBox.min;
    const high = boundingBox.max;
    let listCacuConner = []

    listCacuConner.push(new THREE.Vector3(low.x, low.y, low.z))
    listCacuConner.push(new THREE.Vector3(high.x, low.y, low.z))
    listCacuConner.push(new THREE.Vector3(low.x, high.y, low.z))
    listCacuConner.push(new THREE.Vector3(low.x, low.y, high.z))
    listCacuConner.push(new THREE.Vector3(high.x, high.y, low.z))
    listCacuConner.push(new THREE.Vector3(high.x, low.y, high.z))
    listCacuConner.push(new THREE.Vector3(low.x, high.y, high.z))
    listCacuConner.push(new THREE.Vector3(high.x, high.y, high.z))
    // draw line
    const materialLine = new THREE.LineBasicMaterial({color: 0x000000});
    const points = [];
    // 2 - 6 x
    // 6 - 4 z
    // 6 - 8 y
    const far = 0.5
    const listCorner = [
      {
        index: 1,
        corner: ['2', '6'],
        position: {
          x: ((totalBoundingBox.max.x + totalBoundingBox.min.x) / 2) + far,
          y: Math.abs(totalBoundingBox.max.y + totalBoundingBox.min.y) / -2,
          z: 0
        },
        rotation: {
          x: 4.7,
          y: 0,
          z: 4.7
        }
      },
      {
        index: 2,
        corner: ['6', '4'],
        position: {
          x: 0,
          y: Math.abs(totalBoundingBox.max.y + totalBoundingBox.min.y) / -2,
          z: ((totalBoundingBox.max.x + totalBoundingBox.min.x) / 2) + 0.5
        },
        rotation: {
          x: 4.7,
          y: 0,
          z: 0
        }
      },
      {
        index: 3,
        corner: ['6', '8'],
        position: {
          x: 4.7,
          y: 0,
          z: 0
        }
      }
    ]
    for (const conArray of listCorner) {
      for (const co of conArray.corner) {
        points.push(listCacuConner[parseInt(co) - 1]);
      }
    }

    const geometryLine = new THREE.BufferGeometry().setFromPoints(points);
    const line = new THREE.Line(geometryLine, materialLine);
    scene.add(line);
    // draw text
    var canvas = document.createElement('canvas');
    var context = canvas.getContext('2d');

    function roundUp(numToRound, multiple) {
      var value = multiple;
      while (value < numToRound) {
        value = value * multiple;
      }
      return value;
    }

    const fontSize = distance * 2
    console.log(fontSize)
    const text = '100cm'
    context!.font = fontSize + "px Arial";
    let metrics = context!.measureText(text);

    var textWidth = roundUp(metrics.width + 20.0, 2);
    var textHeight = roundUp(fontSize + 5, 2);

    canvas.width = textWidth;
    canvas.height = textHeight;

    context!.font = fontSize + "px Arial";
    context!.textAlign = "center";
    context!.textBaseline = "middle";
    context!.fillStyle = "#40ed6e";
    context!.fillText(text, textWidth / 2, textHeight / 2);

    var texture = new THREE.Texture(canvas);
    texture.needsUpdate = true;

    var material_text = new THREE.MeshBasicMaterial({
      map: texture,
      transparent: true,
      side: THREE.DoubleSide
    });
    let mesh_text_right = new THREE.Mesh(new THREE.PlaneGeometry(textWidth / 60, textHeight / 60, 10, 10), material_text);
    mesh_text_right.position.y = Math.abs(totalBoundingBox.max.y + totalBoundingBox.min.y) / -2;
    mesh_text_right.position.z = 0;
    mesh_text_right.position.x = ((totalBoundingBox.max.x + totalBoundingBox.min.x) / 2) + 0.5;
    mesh_text_right.rotation.x = 4.7
    mesh_text_right.rotation.z = 4.7
    // let mesh_text_left = new THREE.Mesh(new THREE.PlaneGeometry(textWidth / 60, textHeight / 60, 10, 10), material_text);
    // mesh_text_left.position.y = Math.abs(totalBoundingBox.max.y + totalBoundingBox.min.y) / -2;
    // mesh_text_left.position.x = 0;
    // mesh_text_left.position.z = ((totalBoundingBox.max.x + totalBoundingBox.min.x) / 2) + 0.5;
    // mesh_text_left.rotation.x = 4.7
    // mesh_text_left.rotation.z = 0
    // scene.add(mesh_text_left)
    scene.add(mesh_text_right)


    // nguồn sáng
    const directionalLight = new THREE.DirectionalLight(0xffffff, 2.5);
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
      scene.children[0].children.forEach((e: any) => uuids.push(e.uuid));
      mouse.x =
          (event.clientX /
              (renderer.domElement.clientWidth + (window.innerWidth - renderer.domElement.clientWidth) * 2)) *
          2 -
          1;
      mouse.y = -(event.clientY / renderer.domElement.clientHeight) * 2 + 1;
      raycaster.setFromCamera(mouse, camera);
      let intersects = raycaster.intersectObjects(scene.children[0].children);
      if (intersects.length > 0) {
        for (let i = 0; i < intersects.length; i++) {
          const el = intersects[i];
          if (uuids.findIndex(e => e == el.object.uuid) != -1) {
            if (el.object.type !== "BoxHelper") {
              scene.children[0].children.forEach((item: any) => {
                if (item.type === "Mesh") {
                  item.material.color.set("#ffffff");
                }
              });
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
}

const handleClick = (index: string) => {
  data.value.faces = data.value.faces.map((it: any) => {

    return {
      ...it,
      isShow: index === it.index ? !it.isShow : it.isShow,
    };
  });
  console.log(data.value.faces)
  renderModel3D(data.value)
}

onMounted(async () => {
  // const id = '6dc7533b-90eb-458a-81a5-ce329b40bb1c'
  // const id = 'b6ed3d7a-5e0f-41d2-99e8-51728ad55966'
  const id = 'ccc6a9f8-ef2f-47d8-be19-11dcd092c9e3'
  let res = await axios.get(
      `http://192.168.1.57:8767/analysis/data?id=${id}`
  );
  data.value = res.data.data;
  data.value.faces = data.value.faces.map((it: any) => {
    return {
      ...it,
      isShow: true,
    };
  });
  renderModel3D(data.value)
});
</script>

<template>
  <div style="display: flex;height: 100vh">
    <div
        style="width: 15%; height: 100vh; background-color: #ccc; z-index: 1;overflow-y: auto;"
        id="left"
    >

      <div v-for="item in data.faces" :key="item.index" style="padding: 15px">
        <div>
          <span style="margin-right: 15px">{{ item.index + 1 }}</span>

          <span style="margin-right: 15px">{{
              item.isShow ? "Show" : "Hidden"
            }}</span>

          <button @click="()=>handleClick(item.index)">Change</button>
        </div>
      </div>
    </div>
    <div style="width: 85%; height: 100vh" id="three"></div>
  </div>
</template>
