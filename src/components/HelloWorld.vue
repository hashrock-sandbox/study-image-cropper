<script>
async function loadFromDir(parent, pattern, exclude) {
  const files = [];
  for await (let handle of parent.values()) {
    if (!handle.name.match(exclude) && handle.name.match(pattern)) {
      files.push(handle);
    }
  }
  return files;
}
async function saveImageData(buffer, handle) {
  const handleFile = await handle.getFileHandle("out.png", { create: true });
  const writable = await handleFile.createWritable({
    // keepExistingData: false,
  });
  await writable.write(buffer);
  await writable.close();
}
import Cropper from "cropperjs";
import "cropperjs/dist/cropper.css";

export default {
  data() {
    return {
      msg: "HEllo",
      files: [],
      parent: null,
    };
  },
  methods: {
    async openDir() {
      this.parent = await window.showDirectoryPicker();
      if (!this.parent) {
        return;
      }

      this.files = await loadFromDir(this.parent, /.+\.png/, /.+\.thumb\.png/);
    },
    async createThumb(f) {
      let reader = new FileReader();
      reader.onload = async () => {
        const _result = reader.result;
        const image = new Image();
        image.src = _result;
        const canvas = new OffscreenCanvas(100, 100);
        // canvas.width = 100;
        // canvas.height = 100;
        const ctx = canvas.getContext("2d");
        ctx.imageSmoothingEnabled = true;
        // ctx.translate(0.5, 0.5);
        ctx.drawImage(image, 0, 0, 100, 100);
        // document.body.appendChild(canvas)
        // var imageData = ctx.getImageData(0, 0, 100, 100);
        // var buffer = imageData.data.buffer;
        const blob = await canvas.convertToBlob();
        console.log(blob);
        saveImageData(blob, this.parent);
      };
      reader.readAsDataURL(await f.getFile());
    },
  },
  mounted() {
    const image = document.getElementById("image");
    const cropper = new Cropper(image, {
      aspectRatio: 16 / 9,
      crop(event) {
        console.log(event.detail.x);
        console.log(event.detail.y);
        console.log(event.detail.width);
        console.log(event.detail.height);
        console.log(event.detail.rotate);
        console.log(event.detail.scaleX);
        console.log(event.detail.scaleY);
      },
    });
  },
};
</script>

<template>
  <button @click="openDir">Open Dir</button>
  <div>
    <img id="image" src="https://placekitten.com/g/200/300" />
    <div v-for="(file, i) in files" :key="i">
      {{ file.name }} <button @click="createThumb(file)">Create</button>
    </div>
  </div>
</template>

<style scoped>
a {
  color: #42b983;
}
</style>

