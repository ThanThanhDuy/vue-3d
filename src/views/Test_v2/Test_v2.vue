<script setup lang="ts">
import * as THREE from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
var scene = new THREE.Scene();
var camera = new THREE.PerspectiveCamera(
  45,
  window.innerWidth / window.innerHeight,
  1,
  1000
);
var renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

var geometry1 = new THREE.BoxGeometry(1, 1, 1);
var material1 = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
var cube1 = new THREE.Mesh(geometry1, material1);
scene.add(cube1);

var geometry2 = new THREE.CylinderGeometry(0.5, 0.5, 1, 32);
var material2 = new THREE.MeshBasicMaterial({ color: 0xff0000 });
var cylinder1 = new THREE.Mesh(geometry2, material2);
scene.add(cylinder1);

cube1.position.set(0, 0, 0);
cylinder1.position.set(2, 0, 0);

camera.position.set(0, 0, 5);

var ambientLight = new THREE.AmbientLight(0x404040);
scene.add(ambientLight);

var directionalLight = new THREE.DirectionalLight(0xffffff);
directionalLight.position.set(1, 1, 1).normalize();
scene.add(directionalLight);

function animate() {
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
}

animate();
</script>

<template></template>
