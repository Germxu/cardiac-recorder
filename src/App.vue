<script setup lang="ts">
import { ref, Ref, onMounted, computed } from "vue";
const pipBtnEl = ref<HTMLButtonElement | null>(null) as Ref<HTMLButtonElement>;
const permissionBtnEl = ref<HTMLButtonElement | null>(null) as Ref<HTMLButtonElement>;
const videoEl = ref<HTMLVideoElement | null>(null) as Ref<HTMLVideoElement>;
const canvasEl = ref<HTMLCanvasElement | null>(null) as Ref<HTMLCanvasElement>;

const { setCtx, anim } = drawCanvas(canvasEl);
async function btPermission() {
  try {
    //ts-ignore
    await navigator.bluetooth.requestDevice({
      acceptAllDevices: true,
    });
    console.log("Bluetooth permission granted");

    return { state: "granted" };
  } catch (error) {
    console.error("Bluetooth permission denied");
    return { state: "denied" };
  }
};


onMounted(() => {
  if (!pipBtnEl.value || !videoEl.value || !canvasEl.value) {
    console.error("Element not found");
    return;
  }
  permissionBtnEl.value.onclick = btPermission;

  anim();
  setCtx();
  const stream = canvasEl.value.captureStream();
  videoEl.value.srcObject = stream;

  pipBtnEl.value.onclick = enterPictureInPicture;
});

function drawCanvas(canvasEl: Ref<HTMLCanvasElement>) {
  const ctx = computed(() =>
    canvasEl.value.getContext("2d")
  ) as Ref<CanvasRenderingContext2D>;
  onMounted(() => {
    if (!canvasEl.value) {
      console.error("Element not found");
      return;
    }
    // 设置canvas拖动调节大小
    canvasEl.value.onmousedown = (e) => {
      const rect = canvasEl.value.getBoundingClientRect();
      const x = e.clientX - rect.left;
      const y = e.clientY - rect.top;
      const w = rect.width;
      const h = rect.height;
      if (x > w - 10 && y > h - 10) {
        canvasEl.value.onmousemove = (e) => {
          canvasEl.value.width = e.clientX - rect.left;
          canvasEl.value.height = e.clientY - rect.top;
          setCtx();
        };
        canvasEl.value.onmouseup = () => {
          canvasEl.value.onmousemove = null;
          canvasEl.value.onmouseup = null;
        };
      }
    };
  });

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
      new Date().toTimeString().split(" ")[ 0 ],
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

    pictureInPictureWindow.addEventListener("resize", () => { }, false);
  });
}
</script>

<template>
  <button id="calibrateBtn" ref="permissionBtnEl">Calibrate</button>
  <button id="pipBtn" ref="pipBtnEl">PiP Now</button>
  <div ref="canvas">
    <canvas id="canvas" width="800" height="300" ref="canvasEl"/>
  </div>
  <video id="video" controls muted autoplay ref="videoEl" style="height: 0"/>
</template>
<style>
#calibrateBtn {
  position: fixed;
  top: 2em;
  left: 5em;
}
#pipBtn {
  position: fixed;
  top: 2em;
  right: 2em;
}

#canvas {
  position: relative;
  border: 10px solid black;
}

#canvas::before{
  /* 右上角添加一个handler用于标记可拖动调整尺寸 */
  content: "";
  position: absolute;
  right: 0;
  bottom: 0;
  width: 10px;
  height: 10px;
  background-color: red;
  cursor: nwse-resize;

}
</style>