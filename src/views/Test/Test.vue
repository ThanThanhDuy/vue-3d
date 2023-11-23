<script setup lang="js">
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import axios from "axios"
import {ref} from "vue"
import * as THREE from 'three';

const data = ref()
let controls;
// get data form server
const get_data= async()=>{
  const width = window.innerWidth, height = window.innerHeight;

  var initialZoom = 100;
  const camera = new THREE.OrthographicCamera(
          window.innerWidth / -initialZoom,
          window.innerWidth / initialZoom,
          window.innerHeight / initialZoom,
          window.innerHeight / -initialZoom,
          1,
          1000
        );
 camera.position.z = 0.1;
// const camera = new THREE.PerspectiveCamera( 45, window.innerWidth / window.innerHeight, 0.1, 1000 );

  const scene = new THREE.Scene();

 // do - xanh la cay - vang - ti1m - xanh dam - trang -
  const colors = ['#ff0000','#00ff00','#ffc815','#433e90','#0f8880','#eeeeee','#556f55',]
  const mesh = []
  let res = await axios.get("http://192.168.1.57:8767/analysis/data?id=6dc7533b-90eb-458a-81a5-ce329b40bb1c");
  data.value = res.data.data
  var gridHelper = new THREE.GridHelper(10, 10); // Parameters: size, divisions
   scene.add(gridHelper);

    for (let index = 0; index < data.value.faces.length; index++) {

      const e = data.value.faces[index];
      const geometry = new THREE.BufferGeometry();
      const positions = new Float32Array(e.vertex);
      const normals = new Float32Array(e.normal);
      const indices = new Uint16Array(e.triangle);

      geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
      geometry.setAttribute('normal', new THREE.BufferAttribute(normals, 3));
      geometry.setIndex(new THREE.BufferAttribute(indices, 1));

      // let  p = index>(colors.length-1)? Math.floor(Math.random() * 8):index
      //  p= index ==0 || index ==7 ?0:2;
      var material = new THREE.MeshBasicMaterial({ color: colors[index] });

  // const geometry = new THREE.BoxGeometry( 0.2, 0.2, 0.2 );
  // const material = new THREE.MeshNormalMaterial();

      mesh[index] = new THREE.Mesh( geometry, material )
      scene.add( mesh[index] );
    }

    // for (let index = 0; index < data.value.holes.length; index++) {

    //   const e = data.value.holes[index].children;

    //   for (let j = 0; j < e.length; j++) {
    //     const el = e[j];
    //     console.log(el)
    //     const geometry = new THREE.BufferGeometry();
    //     const positions = new Float32Array(el.vertex);
    //     const normals = new Float32Array(el.normal);
    //     const indices = new Uint16Array(el.triangle);

    //   geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
    //   geometry.setAttribute('normal', new THREE.BufferAttribute(normals, 3));
    //   geometry.setIndex(new THREE.BufferAttribute(indices, 1));

    //   let material = new THREE.MeshBasicMaterial({ color: colors[0] });

    //   let meshs = new THREE.Mesh( geometry, material )
    //   scene.add( meshs );
    //   }


    // }


  function animation( time ) {

    for ( let j = 0 ;j<mesh.length;j++){
        mesh[j].position.x = -2
      //mesh[j].rotation.x = time / 7000;
      //mesh[j].rotation.y = time / 7000;
    }

    renderer.render( scene, camera );
  }

  const renderer = new THREE.WebGLRenderer( ); // { antialias: true }
  let originalSize = new THREE.Vector2();
  renderer.getSize(originalSize)
  console.log( parseInt(originalSize.x,10) )

  renderer.setSize(  width, height );
  // renderer.render(scene, camera);
  renderer.setAnimationLoop( animation );
  document.body.appendChild( renderer.domElement );
  controls = new OrbitControls(camera, renderer.domElement);
		controls.dampingFactor = 50;
		controls.screenSpacePanning = false;
  renderer.domElement.addEventListener('mousemove', onMouseMove);

        function onMouseDown(event) {
        isDragging = true;
        previousMousePosition = {
          x: event.clientX,
          y: event.clientY,
        };
      }

      function onMouseUp() {
        isDragging = false;
      }

      function onMouseMove(event) {
        if (!isDragging) return;

var deltaMove = {
  x: event.clientX - previousMousePosition.x,
  y: event.clientY - previousMousePosition.y,
};

cube.rotation.x += deltaMove.y * 0.01;
cube.rotation.y += deltaMove.x * 0.01;

previousMousePosition = {
  x: event.clientX,
  y: event.clientY,
};
      }
  window.addEventListener('keydown', function (event) {

          if (event.key === '+') {
            camera.zoom *= 1.1;
            camera.updateProjectionMatrix();
          } else if (event.key === '-') {
            camera.zoom /= 1.1;
            camera.updateProjectionMatrix();
          }
        });

}

get_data();


console.log(new Date().getTime())
</script>
<template>
  <div class="testclass">This is a div</div>
</template>

<style scoped>
.test {
}
</style>
