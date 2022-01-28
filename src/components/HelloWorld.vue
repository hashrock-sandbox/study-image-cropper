<template>
  <button @click="openDir">Open Dir</button>
  <img id="image" />
  <div v-if="editing">
    <div>
      <button @click="crop">Save Thumbnail</button>
    </div>
  </div>

  <div>
    <div class="item" v-for="(file, i) in files" :key="i" @click="loadCropper(file)">
      {{ file.name }} 
      <span v-if="hasThumbnail.includes(file.name)">✔</span>
    </div>
  </div>
</template>

<script>
import Cropper from "cropperjs";
import "cropperjs/dist/cropper.css";

async function loadFromDir(parent, pattern, exclude) {
  const files = [];
  for await (let handle of parent.values()) {
    if(exclude){
      if (!handle.name.match(exclude) && handle.name.match(pattern)) {
        files.push(handle);
      }
    }else{
      if (handle.name.match(pattern)) {
        files.push(handle);
      }
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

let cropper = null
export default {
  data() {
    return {
      files: [],
      parent: null,
      editing: null,
      thumbs: []
    };
  },
  methods: {
    async findThumbs(){
      this.thumbs = await loadFromDir(this.parent, /.+\.thumb\.png/);
    },
    async openDir() {
      this.parent = await window.showDirectoryPicker();
      if (!this.parent) {
        return;
      }
      this.files = await loadFromDir(this.parent, /.+\.png/, /.+\.thumb\.png/);
      this.findThumbs()
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
      }).toBlob(async (blob)=>{
        await saveImageData(blob, this.parent, this.editing.name.replace(/\.png/, ".thumb.png"))
        await this.findThumbs()
      })
    }
  },
  computed: {
    hasThumbnail(){
      return this.thumbs.map(i => i.name.replace(".thumb.png", ".png"))
    }
  },
  mounted() {
    const image = document.getElementById("image");
    cropper = new Cropper(image, {
      aspectRatio: 1,
      crop(event) {
      },
    });
  },
};
</script>

<style scoped>
.item{
  cursor: pointer;
}
.item:hover{
  background: #AAA;
}
a {
  color: #42b983;
}
img {
  display: block;
  max-width: 100%;
  max-height: 400px;
}
</style>

