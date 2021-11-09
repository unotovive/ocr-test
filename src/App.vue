<template>
  <div id="app">
    <div>
      <video ref="videoRef" autoPlay style="width: 100%" />
      <pre>
        <h1>{{ text }}</h1>
      </pre>
    </div>
  </div>
</template>

<script>
import { createWorker } from "tesseract.js";

export default {
  name: "App",
  data() {
    return {
      worker: null,
      stream: null,
      text: "",
    };
  },
  async mounted() {
    const worker = createWorker();
    await worker.load();
    await worker.loadLanguage("eng");
    await worker.initialize("eng");
    this.worker = worker;

    const stream = await navigator.mediaDevices.getUserMedia({
      video: { width: 1000, height: 1000 },
      audio: false,
    });
    this.stream = stream;
  },
  watch: {
    worker() {
      if (this.worker && this.stream && this.$refs.videoRef.current !== null) {
        this.$refs.videoRef.srcObject = this.stream;
        const clear = this.onRecognizeText();
        return clear;
      }
    },
    stream() {
      if (this.worker && this.stream && this.$refs.videoRef.current !== null) {
        this.$refs.videoRef.srcObject = this.stream;
        const clear = this.onRecognizeText();
        return clear;
      }
    },
  },
  methods: {
    onRecognizeText() {
      const timerId = setInterval(async () => {
        if (this.$refs.videoRef === null || !this.worker) return;

        const c = document.createElement("canvas");
        c.width = 1000;
        c.height = 1000;
        c.getContext("2d")?.drawImage(this.$refs.videoRef, 0, 0, 1000, 1000);
        const {
          data: { text },
        } = await this.worker.recognize(c);
        this.text = text;
      }, 500);
      return () => clearInterval(timerId);
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
