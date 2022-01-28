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
async function saveImageData(buffer, handle, filename) {
  const handleFile = await handle.getFileHandle(filename, { create: true });
  const writable = await handleFile.createWritable({
    // keepExistingData: false,
  });
  await writable.write(buffer);
  await writable.close();
}
import Cropper from "cropperjs";
import "cropperjs/dist/cropper.css";


let cropper = null
export default {
  data() {
    return {
      msg: "HEllo",
      files: [],
      parent: null,
      editing: null
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
    async loadCropper(f){
      let reader = new FileReader();
      reader.onload = async () => {
        const _result = reader.result;
        const image = new Image();
        image.src = _result;
        cropper.replace(_result)
        this.editing = f
      };
      reader.readAsDataURL(await f.getFile());
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
    crop(){
      // 何か解像度が低い
      // https://github.com/fengyuanchen/cropperjs/issues/372
      cropper.getCroppedCanvas({
        width: 300,
        height: 300,
        minWidth: 300,
        minHeight: 300,
        maxWidth: 4096,
        maxHeight: 4096,
        fillColor: '#fff',
        imageSmoothingEnabled: true,
        imageSmoothingQuality: 'high',
      }).toBlob((blob)=>{
        saveImageData(blob, this.parent, this.editing.name.replace(/\.png/, ".thumb.png"))
      })
    }
  },
  mounted() {
    const image = document.getElementById("image");
    cropper = new Cropper(image, {
      aspectRatio: 1,
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
    <img id="image" />
    <div>
      <button @click="crop">Crop</button>
    </div>
  </div>

  <div>
    <div v-for="(file, i) in files" :key="i">
      {{ file.name }} 
      <button @click="createThumb(file)">Create</button>
      <button @click="loadCropper(file)">Load</button>
    </div>
  </div>
</template>

<style scoped>
a {
  color: #42b983;
}
img {
  display: block;
  max-width: 100%;
  /* max-height: 400px; */
}
</style>

