<template>
  <div class="file-pane">
    <button @click="openDir">Open Dir</button>
    <div>
      <div class="item" v-for="(file, i) in files" :key="i" @click="loadCropper(file)">
        {{ file.name }} 
        <span v-if="hasThumbnail.includes(file.name)">✔</span>
      </div>
    </div>
  </div>
  <div class="image-pane">
    <div>
      <div v-if="editing">
        <button @click="crop">Save Thumbnail</button>
      </div>
    </div>
    <img id="image" />
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
      this.thumbs = await loadFromDir(this.parent, /.+\.thumb\.webp/);
    },
    async openDir() {
      this.parent = await window.showDirectoryPicker();
      if (!this.parent) {
        return;
      }
      this.files = await loadFromDir(this.parent, /.+\.(png|jpg|jpeg)/, /.+\.thumb\.webp/);
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
        width: 600,
        height: 600,
        minWidth: 600,
        minHeight: 600,
        maxWidth: 4096,
        maxHeight: 4096,
        fillColor: '#fff',
        imageSmoothingEnabled: true,
        imageSmoothingQuality: 'high',
      }).toBlob(async (blob)=>{
        await saveImageData(blob, this.parent, this.editing.name + ".thumb.webp")
        await this.findThumbs()
      }, "image/webp", 0.7)
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
  padding: 0.25em 0.5em;
  font-size: 14px;
  color: rgb(64, 99, 95);
}
.item:hover{
  background: rgb(184, 202, 196);
}
a {
  color: #42b983;
}
img {
  display: block;
  max-width: 100%;
  max-height: 400px;
}
.file-pane{
  width: 300px;
}
.image-pane{
  flex: 1;
  background: #333;
}
</style>

