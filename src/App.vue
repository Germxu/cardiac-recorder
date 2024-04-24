<script setup lang="ts">
import { ref, Ref, onMounted, computed } from "vue";
const btnEl = ref<HTMLButtonElement | null>(null) as Ref<HTMLButtonElement>;
const videoEl = ref<HTMLVideoElement | null>(null) as Ref<HTMLVideoElement>;
const canvasEl = ref<HTMLCanvasElement | null>(null) as Ref<HTMLCanvasElement>;
const ctx = computed(() =>
  canvasEl.value.getContext("2d")
) as Ref<CanvasRenderingContext2D>;

const { setCtx, anim } = drawCanvas(ctx);

onMounted(() => {
  if (!btnEl.value || !videoEl.value || !canvasEl.value) {
    console.error("Element not found");
    return;
  }

  anim();
  setCtx();
  const stream = canvasEl.value.captureStream();
  videoEl.value.srcObject = stream;

  btnEl.value.onclick = enterPictureInPicture;
});

function drawCanvas(ctx: Ref<CanvasRenderingContext2D>) {
  function setCtx() {
    ctx.value.font = "180px Arial";
    ctx.value.textAlign = "center";
    ctx.value.textBaseline = "middle";
  }

  function anim() {
    ctx.value.fillStyle = "white";
    ctx.value.fillRect(0, 0, canvasEl.value.width, canvasEl.value.height);
    ctx.value.fillStyle = "black";
    ctx.value.fillText(
      new Date().toTimeString().split(" ")[0],
      canvasEl.value.width / 2,
      canvasEl.value.height / 2
    );
    requestAnimationFrame(anim);
  }
  return { setCtx, anim };
}
function enterPictureInPicture() {
  videoEl.value.requestPictureInPicture().then((pictureInPictureWindow) => {
    console.log("Entered Picture-in-Picture mode.");

    pictureInPictureWindow.addEventListener("resize", () => {}, false);
  });
}
</script>

<template>
  <canvas id="canvas" width="800" height="300" ref="canvasEl"></canvas>
  <video
    id="video"
    controls
    muted
    autoplay
    ref="videoEl"
    style="height: 0"
  ></video>
  <button id="btn" ref="btnEl">PiP Now</button>
</template>
