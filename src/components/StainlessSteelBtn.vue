<script setup>
import { onMounted, ref } from "vue";

let mediaStreamTrack = {};
let canvas = null;
let ctx = null;

onMounted(() => {
  getCamera();
});

const getCamera = () => {
  // 获取 canvas 画布
  canvas = document.createElement("canvas");
  ctx = canvas.getContext("2d");
  // 旧版本浏览器可能根本不支持mediaDevices，我们首先设置一个空对象
  if (navigator.mediaDevices === undefined) {
    navigator.mediaDevices = {};
  }
  // 正常支持版本
  navigator.mediaDevices
    .getUserMedia({
      video: true,
    })
    .then((stream) => {
      // 摄像头开启成功
      mediaStreamTrack =
        typeof stream.stop === "function" ? stream : stream.getTracks()[0];

      const video = document.createElement("video");
      video.srcObject = stream;
      video.play();

      // 将图像绘制到 canvas 上
      function update() {
        const imageCapture = new ImageCapture(mediaStreamTrack);
        imageCapture
          .grabFrame()
          .then((imageBitmap) => {
            // 将图像绘制到 canvas 上

            canvas.width = imageBitmap.width;
            canvas.height = imageBitmap.height;
            // const ctx = canvas.getContext("2d");

            // 将图像水平翻转，实现镜像效果
            ctx.scale(-1, 1);
            ctx.translate(-canvas.width, 0);
            ctx.filter = "blur(3px)";
            // 绘制图像中心位置
            const centerX = 100;
            const centerY = 500;

            // 只绘制中间部分
            const cropWidth = 400;
            const cropHeight = 600;
            const cropLeft = centerX - cropWidth / 4;
            const cropTop = centerY - cropHeight / 4;

            ctx.drawImage(
              imageBitmap,
              cropLeft,
              cropTop,
              cropWidth + 10,
              cropHeight,
              50,
              -6,
              canvas.width,
              canvas.height
            );

            // 将 canvas 转换为 data URL，并将其设置为按钮的背景
            const dataURL = canvas.toDataURL("image/jpeg");
            btn.style.backgroundImage = `url(${dataURL})`;

            requestAnimationFrame(update);
          })
          .catch((error) => {
            if (error == undefined) requestAnimationFrame(update);
          });
      }
      requestAnimationFrame(update);
    })
    .catch((err) => {
      console.log(err);
    });
};
</script>

<template>
  <button id="btn">Button</button>
</template>

<style scoped>
button {
  width: 300px;
  height: 100px;
  font-size: 40px;
  border-radius: 100px;
  border-top: 1px solid rgba(255, 255, 255, 0.9);
  border-left: 2px solid rgba(255, 253, 253, 0.776);
  border-right: 2px solid rgba(255, 253, 253, 0.7);
  border-bottom: 2px solid rgba(255, 253, 253, 0.7);
  display: block;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: #808080;
  box-shadow: 8px 0px 40px rgba(0, 0, 0, 0.8);
  font-weight: bold;
  text-shadow: 0px 1px 1px #ffffff, 0px 1px 1px #000000ba;
  color: rgba(14, 14, 14, 0.915);
}
</style>
