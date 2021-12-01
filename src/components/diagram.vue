<!-- 模仿github代码提交热力图 -->
<script setup lang="ts">
import { reactive, ref, onMounted } from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';

const init = () => {
  // 创建场景
  const scene = new THREE.Scene();
  // 创建光源
  const camera = new THREE.PerspectiveCamera(
    25,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );

  // 创建构造器
  const renderer = new THREE.WebGLRenderer();
  renderer.setClearColor(new THREE.Color(0xffffff));

  let axes = new THREE.AxesHelper(20);
  // 坐标系添加在场景中
  scene.add(axes);
  renderer.setSize(window.innerWidth, window.innerHeight);

  // 创建一个平面
  let planeGeometry = new THREE.PlaneGeometry(20, 100);
  let planeMaterial = new THREE.MeshBasicMaterial({ color: 0xcccccc });
  let plane = new THREE.Mesh(planeGeometry, planeMaterial);
  plane.rotation.x = -0.5 * Math.PI;
  plane.position.x = 0;
  plane.position.y = 0;
  plane.position.z = 0;
  scene.add(plane);

  // 定位相机 并且指向中心
  camera.position.x = -100;
  camera.position.y = 50;
  camera.position.z = 95;

  camera.lookAt(scene.position);
  // 创建一个正方体
  const colorList = [0xebedf0, 0x9be9a8, 0x40c463, 0x30a14e, 0x216e39];
  const list = [1, 2, 3, 4, 2, 4, 2];

  const arr = [
    [1, 1, 3, 0, 0, 1, 0],
    [2, 4, 2, 1, 2, 1, 1],
    [1, 0, 3, 0, 0, 2, 2],
    [1, 2, 3, 1, 0, 2, 1],
    [2, 0, 2, 4, 2, 1, 1],
    [1, 2, 3, 4, 0, 2, 2],
    [0, 0, 3, 1, 0, 4, 0],
    [2, 4, 2, 1, 2, 1, 1],
    [1, 2, 3, 2, 0, 2, 0],
    [0, 2, 3, 1, 2, 4, 2],
    [2, 4, 2, 4, 0, 1, 1],
    [1, 2, 0, 1, 2, 4, 2],
    [0, 2, 3, 0, 1, 0, 0],
    [2, 1, 2, 1, 0, 1, 1],
    [1, 2, 3, 4, 2, 4, 0],
    [2, 4, 2, 1, 2, 1, 1],
    [1, 0, 3, 0, 0, 2, 2],
    [1, 2, 3, 1, 0, 2, 0],
    [2, 0, 2, 4, 2, 1, 1],
    [1, 2, 0, 1, 0, 2, 2],
    [0, 0, 0, 1, 2, 4, 0],
    [2, 4, 2, 1, 2, 1, 1],
    [1, 2, 3, 2, 0, 2, 0],
    [0, 2, 3, 1, 2, 4, 2],
    [2, 4, 2, 4, 0, 1, 1],
    [1, 2, 0, 1, 2, 4, 2],
    [0, 2, 3, 0, 1, 0, 0],
    [2, 4, 2, 2, 0, 1, 1],
    [1, 2, 3, 4, 0, 4, 0],
  ];

  arr.forEach((item1, index1) => {
    item1.forEach((item2, index2) => {
      const boxgeometry = new THREE.BoxGeometry(1, item2, 1);
      const boxmaterial = new THREE.MeshPhongMaterial({
        color: colorList[item2],
        flatShading: true, // 有棱角的
      });
      // const boxmaterial = new THREE.MeshBasicMaterial({
      //   color: colorList[item2],
      // });
      const box = new THREE.Mesh(boxgeometry, boxmaterial);
      box.position.y = item2 / 2;
      box.position.z = 0.5 + index1 * 1.2;
      box.position.x = 0.5 + index2 * 1.2;
      box.castShadow = true;
      scene.add(box);
    });
  });

  var spotLight = new THREE.SpotLight(0xffffff);
  spotLight.position.set(130, 130, -130);
  spotLight.castShadow = true;
  spotLight.angle = Math.PI / 4;
  spotLight.shadow.mapSize.width = 1024;
  // 添加聚光灯
  scene.add(spotLight);

  renderer.render(scene, camera);
  let dom = document.getElementById('webgl-output') as HTMLElement;
  dom.appendChild(renderer.domElement);

  var controls = new OrbitControls(camera, renderer.domElement);
  // 监听控制器的鼠标事件，执行渲染内容
  controls.addEventListener('change', () => {
    renderer.render(scene, camera);
  });
};

onMounted(() => {
  console.log('start');
  init();
});
</script>

<template>
  <div class="container">
    <!-- <h3>代码提交热力图</h3> -->
    <div id="webgl-output"></div>
  </div>
</template>

<style lang="scss" scoped></style>
