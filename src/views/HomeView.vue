<template>
  <div class="container" ref="container"></div>
</template>

<script setup>
import { onMounted, onUnmounted, ref } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import gsap from "gsap";

let first = true;
let videoIndex = 0;
let runing = false;
let videoList = [
  "/src/assets/video/Code.mov",
  "/src/assets/video/Earth.mov",
  "/src/assets/video/Ocean.mov",
];
const container = ref(null);
onMounted(() => {
  // 初始化场景
  const scene = new THREE.Scene();
  // 初始化相机
  const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );
  // 初始化相机位置
  camera.position.set(0, 0, 6);
  camera.aspect = window.innerWidth / window.innerHeight;
  // 更新相机矩阵
  camera.updateProjectionMatrix();

  // 加载背景纹理
  const loader = new THREE.TextureLoader();
  // loader.load("/src/assets/images/background.jpg", (texture) => {
  //   scene.background = texture;
  // });

  // 加载视频纹理
  let video = document.createElement("video");
  video.src = videoList[videoIndex];
  video.muted = true;
  video.loop = true;
  video.autoplay = "autoplay";
  const videoTexture = new THREE.VideoTexture(video);
  videoTexture.wrapS = videoTexture.wrapT = THREE.ClampToEdgeWrapping;
  videoTexture.minFilter = THREE.LinearFilter;

  // 加载地球纹理
  const earthTexture = loader.load("/src/assets/images/earth.jpg");

  // 添加环境光
  const ambient = new THREE.AmbientLight(0xffffff, 1);
  scene.add(ambient);
  // 添加物体
  const box = new THREE.Mesh(
    new THREE.BoxGeometry(2, 2, 2),
    new THREE.MeshPhysicalMaterial({
      color: 0xffffff,
      transmission: 1.5,
      roughness: 0,
      thickness: 0.5,
      ior: 1.5,
    })
  );
  box.rotateX(Math.PI / 8);
  box.rotateY(-Math.PI / 8);
  const round = new THREE.Mesh(
    new THREE.SphereGeometry(1, 32, 16),
    new THREE.MeshBasicMaterial({
      color: 0xffffff,
      map: earthTexture,
    })
  );
  round.position.set(0, 0, 0);
  box.add(round);
  scene.add(box);
  // 初始化渲染器
  const renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setClearColor(0xffffff, 1);
  // 监听浏览器大小变化
  window.addEventListener("resize", () => {
    renderer.setSize(window.innerWidth, window.innerHeight);
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
  });
  // 将渲染器添加到页面
  container.value.appendChild(renderer.domElement);
  // 添加控制器
  // const controls = new OrbitControls(camera, renderer.domElement);
  // controls.enableDamping = true;
  // 创建投射光线对象
  const raycaster = new THREE.Raycaster();
  // 鼠标位置对象
  const mouse = new THREE.Vector2();
  // 监听鼠标点击事件
  window.addEventListener("click", async (event) => {
    mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
    mouse.y = (event.clientY / window.innerHeight) * 2 - 1;
    let changeIndex = event.clientX >= window.innerWidth / 2 ? 1 : -1;
    raycaster.setFromCamera(mouse, camera);
    let result = raycaster.intersectObject(box, false)[0];
    let ratio = window.innerWidth / window.innerHeight;
    let scaleValue = 2.61;
    if (runing) return;
    runing = true;
    if (result && first) {
      box.remove(round);
      box.material = new THREE.MeshPhysicalMaterial({
        color: 0xffffff,
        transmission: 1.5,
        roughness: 0,
        thickness: 0.5,
        ior: 1.5,
        map: videoTexture,
      });
      let timeLine = gsap.timeline({ onComplete: () => (runing = false) });
      timeLine.to(box.rotation, { x: 0, y: Math.PI, duration: 0.8 });
      timeLine.to(
        box.scale,
        {
          x: scaleValue,
          y: scaleValue,
          z: scaleValue,
          duration: 0.8,
        },
        "<"
      );
      timeLine.to(box.scale, {
        x: scaleValue * ratio,
        duration: 0.2,
        ease: "circ.out",
      });
      first = false;
    } else if (result) {
      let timeLine = gsap.timeline({ onComplete: () => (runing = false) });
      timeLine.to(box.scale, {
        x: scaleValue,
        duration: 0.2,
        ease: "circ.out",
      });
      timeLine.to(box.rotation, {
        x: Math.PI / 8,
        y: -Math.PI / 8,
        duration: 0.8,
      });
      timeLine.to(
        box.scale,
        {
          x: 1,
          y: 1,
          z: 1,
          duration: 0.8,
        },
        "<"
      );
      timeLine.to(box.rotation, {
        x: 0,
        y: Math.PI,
        duration: 0.8,
        onStart: () => tlComplete(changeIndex),
      });
      timeLine.to(
        box.scale,
        {
          x: scaleValue,
          y: scaleValue,
          z: scaleValue,
          duration: 0.8,
        },
        "<"
      );
      timeLine.to(box.scale, {
        x: scaleValue * ratio,
        duration: 0.2,
        ease: "circ.out",
      });
    }
    function tlComplete(index) {
      let nextIndex = videoIndex + index;
      if (nextIndex < 0) {
        videoIndex = videoList.length - 1;
      } else if (nextIndex > videoList.length - 1) {
        videoIndex = 0;
      } else {
        videoIndex = nextIndex;
      }
      video.src = videoList[videoIndex];
    }
  });
  let roundAnimation = gsap.to(round.rotation, {
    duration: 16,
    ease: "linear",
    repeat: -1,
    y: Math.PI * 2,
  });
  // 渲染函数
  const render = () => {
    // controls.update();
    renderer.render(scene, camera);
    requestAnimationFrame(render);
  };
  render();
});
</script>

<style lang="scss" scoped></style>
