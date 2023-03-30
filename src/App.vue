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
  container.value.appendChild(renderer.domElement);
  render();
  watchMouseEvent();

  // 创建客厅
  const livingroom = new Room("客厅", 0, "./img/livingroom/");
  const kitchenPosition = new THREE.Vector3(0, 0, -10);
  const kitchen = new Room("厨房", 3, "./img/kitchen/", kitchenPosition);

  // 创建厨房精灵文字
  let kitchenTextPosition = new THREE.Vector3(0, 0, -5);
  let kitchenText = new SpriteText("厨房", kitchenTextPosition);
  kitchenText.onclick(() => {
    // 让相机移动到厨房
    console.log("厨房");
  });
});
const render = () => {
  renderer.render(scene, camera);
  requestAnimationFrame(render);
};
const watchMouseEvent = () => {
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
};
class SpriteText {
  constructor(text = "", position = new THREE.Vector3(0, 0, 0)) {
    this.callbacks = [];
    const canvas = document.createElement("canvas");
    canvas.width = 1024;
    canvas.height = 1024;
    const context = canvas.getContext("2d");
    context.fillStyle = "rgba(100,100,100,0.7)";
    context.fillRect(0, 256, 1024, 512);
    context.textAlign = "center";
    context.textBaseline = "middle";
    context.font = "bold 200px Arial";
    context.fillStyle = "white";
    context.fillText(text, 512, 512);
    let texture = new THREE.CanvasTexture(canvas);
    const material = new THREE.SpriteMaterial({
      map: texture,
      transparent: true,
    });
    const sprite = new THREE.Sprite(material);
    this.sprite = sprite;
    sprite.position.copy(position);
    scene.add(sprite);

    let mouse = new THREE.Vector2();
    let raycaster = new THREE.Raycaster();
    window.addEventListener("click", (event) => {
      mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
      mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
      raycaster.setFromCamera(mouse, camera);
      let intersects = raycaster.intersectObject(sprite);
      if (intersects.length) {
        this.callbacks.forEach((callback) => {
          callback();
        });
      }
    });
  }
  onclick(callback) {
    this.callbacks.push(callback);
  }
}
class Room {
  constructor(
    name,
    roomIndex,
    textureUrl,
    position = new THREE.Vector3(0, 0, 0),
    euler = new THREE.Euler(0, 0, 0)
  ) {
    this.name = name;
    // 创建立方体
    const geometry = new THREE.BoxGeometry(10, 10, 10);
    geometry.scale(1, 1, -1);

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
      const texture = new THREE.TextureLoader().load(
        textureUrl + item + ".jpg"
      );
      boxMaterials.push(new THREE.MeshBasicMaterial({ map: texture }));
    });
    // 创建物体
    const cube = new THREE.Mesh(geometry, boxMaterials);
    cube.position.copy(position);
    cube.rotation.copy(euler);
    scene.add(cube);
  }
}
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
