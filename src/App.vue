<template>
  <div class="container" ref="container"></div>
</template>

<script setup>
import * as THREE from "three";
import { onMounted, ref } from "vue";

// 初始化场景
const scene = new THREE.Scene();
// 初始化相机
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);

camera.position.set(0, 0, 0);

const container = ref(null);
// 初始化渲染器
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);

// 创建辅助坐标轴
const axes = new THREE.AxesHelper(5);
scene.add(axes);

onMounted(() => {
  // 创建立方体
  const geometry = new THREE.BoxGeometry(500, 500, 500);
  geometry.scale(1, 1, -1);

  let textureUrl = "./img/livingroom/";
  let roomIndex = 0;
  let arr = [
    `${roomIndex}_l`,
    `${roomIndex}_r`,
    `${roomIndex}_u`,
    `${roomIndex}_d`,
    `${roomIndex}_b`,
    `${roomIndex}_f`,
  ];
  let boxMaterials = [];
  arr.forEach((item) => {
    // 创建纹理加载
    const texture = new THREE.TextureLoader().load(textureUrl + item + ".jpg");
    boxMaterials.push(new THREE.MeshBasicMaterial({ map: texture }));
  });
  // 创建物体
  const cube = new THREE.Mesh(geometry, boxMaterials);
  scene.add(cube);

  function render() {
    renderer.render(scene, camera);
    requestAnimationFrame(render);
  }
  container.value.appendChild(renderer.domElement);
  render();

  let isMouseDown = false;
  // 监听鼠标按下事件
  container.value.addEventListener(
    "mousedown",
    () => {
      isMouseDown = true;
    },
    false
  );
  container.value.addEventListener(
    "mouseup",
    () => {
      isMouseDown = false;
    },
    false
  );
  container.value.addEventListener(
    "mouseout",
    () => {
      isMouseDown = false;
    },
    false
  );
  // 是否按下鼠标 移动鼠标
  container.value.addEventListener("mousemove", (event) => {
    if (isMouseDown) {
      camera.rotation.y += event.movementX * 0.001;
      camera.rotation.x += event.movementY * 0.001;
      camera.rotation.order = "YXZ";
    }
  });
});
</script>

<style>
* {
  padding: 0;
  margin: 0;
}
.container {
  width: 100vw;
  height: 100vh;
}
</style>
