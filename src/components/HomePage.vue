<template>
  <section>
    <div class="container">
      <div>
        <h1>Detectly</h1>
        <small>Detects object scanned through user camera</small>
      </div>
      <div class="d-flex flex-column justify-content-center align-items-center">
        <div>
          <button
            class="btn btn-primary my-3"
            v-if="!isStreaming"
            @click="openCamera"
          >
            Open Camera
          </button>
          <div v-else>
            <button class="btn btn-danger m-3" @click="stopStreaming">
              Stop Streaming
            </button>
            <button class="btn btn-primary m-3" @click="snapshot">
              Snapshot
            </button>
          </div>
        </div>
        <video ref="videoRef" autoplay="true" width="200" />
        <img
          class="m-3 "
          ref="imgRef"
          src="https://images.unsplash.com/photo-1582079313048-f13462814383?ixid=MXwxMjA3fDB8MHxzZWFyY2h8MTJ8fGRvZ3xlbnwwfDJ8MHw%3D&ixlib=rb-1.2.1&auto=format&fit=crop&w=500&q=60"
          width="300"
          crossorigin="anonymous"
        />
        <div>
          <button class="btn btn-info my-3" @click="detect">
            <span v-if="isLoading">Loading ... </span>
            <span v-else>Detect Object</span>
          </button>
          <div v-if="result.length > 0">
            <p class="bg-success">{{ result[0].class }}</p>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import { ref } from "vue";
require("@tensorflow/tfjs-backend-cpu");
require("@tensorflow/tfjs-backend-webgl");
const cocoSsd = require("@tensorflow-models/coco-ssd");
export default {
  setup() {
    const imgRef = ref("");
    const videoRef = ref("");
    const isLoading = ref(false);
    const isStreaming = ref(false);
    const result = ref([]);

    // detects the image using classification model
    async function detect() {
      const img = imgRef.value;
      isLoading.value = true;
      const model = await cocoSsd.load();
      const predictions = await model.detect(img);
      result.value = predictions;
      isLoading.value = false;
    }
    //opens camera from the browser
    async function openCamera() {
      if (navigator.mediaDevices.getUserMedia) {
        const devices = await navigator.mediaDevices.enumerateDevices();
        const cams = devices.filter((device) => device.kind === "videoinput");
        const camId = cams[0].deviceId;
        const video = cams.length === 1 ? true : { deviceId: { exact: camId } };
        navigator.mediaDevices.getUserMedia({ video: video }).then((stream) => {
          isStreaming.value = true;
          videoRef.value.srcObject = stream;
        });
      }
    }
    // stops streaming
    function stopStreaming() {
      const stream = videoRef.value.srcObject;
      const tracks = stream.getTracks();
      tracks.map((track) => track.stop());
      isStreaming.value = false;
    }

    // takes picture
    function snapshot() {
      const canvas = document.createElement("canvas");
      const ctx = canvas.getContext("2d");
      ctx.drawImage(videoRef.value, 0, 0, 200, 200);
      const data = canvas.toDataURL("image/png");
      imgRef.value.setAttribute("src", data);
    }
    return {
      imgRef,
      result,
      detect,
      isLoading,
      openCamera,
      videoRef,
      isStreaming,
      stopStreaming,
      snapshot,
    };
  },
};
</script>
