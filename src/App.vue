<template>
  <div class="container" ref="container"></div>
  <div class="map">
    <div class="tag" ref="tagDiv"></div>
    <img src="./assets/map.gif" alt="" />
  </div>
</template>

<script setup>
import * as THREE from "three";
import { onMounted, ref } from "vue";
import gsap from "gsap";
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

const tagDiv = ref(null);
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
  tagDiv.value.style.transform = `translate(100px,110px)`;

  // 创建客厅
  const livingroom = new Room("客厅", 0, "./img/livingroom/");
  const kitchenPosition = new THREE.Vector3(-5, 0, -10);
  const kitchenEuler = new THREE.Euler(0, -Math.PI / 2, 0);
  const kitchen = new Room(
    "厨房",
    3,
    "./img/kitchen/",
    kitchenPosition,
    kitchenEuler
  );

  // 创建厨房精灵文字
  let kitchenTextPosition = new THREE.Vector3(-1, 0, -3);
  let kitchenText = new SpriteText("厨房", kitchenTextPosition);
  kitchenText.onclick(() => {
    // 让相机移动到厨房
    gsap.to(camera.position, {
      duration: 1,
      x: kitchenPosition.x,
      y: kitchenPosition.y,
      z: kitchenPosition.z,
    });
    moveTag("厨房");
  });

  // 创建厨房回客厅精灵文字
  let kitchenBackTextPosition = new THREE.Vector3(-4, 0, -6);
  let kitchenBackText = new SpriteText("客厅", kitchenBackTextPosition);
  kitchenBackText.onclick(() => {
    // 让相机移动到厨房
    gsap.to(camera.position, {
      duration: 1,
      x: 0,
      y: 0,
      z: 0,
    });
    moveTag("客厅");
  });

  // 创建阳台
  let balconyPosition = new THREE.Vector3(0, 0, 15);
  let blalcony = new Room("阳台", 8, "./img/balcony/", balconyPosition);
  // 创建阳台精灵文字
  let blalconyTextPosition = new THREE.Vector3(0, 0, 3);
  let blalconyText = new SpriteText("阳台", blalconyTextPosition);
  blalconyText.onclick(() => {
    // 让相机移动到厨房
    gsap.to(camera.position, {
      duration: 1,
      x: balconyPosition.x,
      y: balconyPosition.y,
      z: balconyPosition.z,
    });
    moveTag("阳台");
  });

  let blalconyBackTextPosition = new THREE.Vector3(-1, 0, 12);
  let blalconyBackText = new SpriteText("客厅", blalconyBackTextPosition);
  blalconyBackText.onclick(() => {
    // 让相机移动到厨房
    gsap.to(camera.position, {
      duration: 1,
      x: 0,
      y: 0,
      z: 0,
    });
    moveTag("客厅");
  });
});
const moveTag = (name) => {
  let positions = {
    客厅: [100, 110],
    厨房: [180, 190],
    阳台: [50, 50],
  };
  console.log(positions[name]);
  if (positions[name]) {
    const target = positions[name];
    gsap.to(tagDiv.value, {
      duration: 0.5,
      x: target[0],
      y: target[1],
      ease: "power3.inOut",
    });
  }
};
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
    sprite.scale.set(0.5, 0.5, 0.5);
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
      if (item.includes("d") || item.includes("u")) {
        texture.rotation = Math.PI;
        texture.center = new THREE.Vector2(0.5, 0.5);
      }
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

.map {
  width: 300px;
  height: 260px;
  position: absolute;
  left: 0;
  bottom: 0;
  overflow: hidden;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
}
.map > img {
  width: 100%;
  height: 100%;
  position: absolute;
  left: 0;
  top: 0;
}
.map > .tag {
  position: absolute;
  top: 0;
  left: 0;
  width: 30px;
  height: 30px;
  background-image: url(./assets/location.png);
  background-size: cover;
  z-index: 1;
}
.loading {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-image: url(./assets/loading.png);
  background-size: cover;
  filter: blur(50px);
  z-index: 100;
}
.progress {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 101;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 20px;
  color: #fff;
}
.progress > img {
  padding: 0 15px;
}

.title {
  width: 180px;
  height: 40px;
  position: fixed;
  right: 100px;
  top: 50px;
  background-color: rgba(0, 0, 0, 0.5);
  line-height: 40px;
  text-align: center;
  color: #fff;
  border-radius: 5px;
  z-index: 110;
}
</style>
