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
        const canvas = document.createElement("canvas");
        canvas.width = 100;
        canvas.height = 100;
        const ctx = canvas.getContext("2d");
        ctx.drawImage(image, 0, 0, 100, 100);
        document.body.appendChild(canvas)
        // var imageData = ctx.getImageData(0, 0, 100, 100);
        // var buffer = imageData.data.buffer;
        canvas.toBlob((blob)=>{
          saveImageData(blob, this.parent);
        })
      };
      reader.readAsDataURL(await f.getFile());
    },
  },
};
</script>

<template>
  <button @click="openDir">Open Dir</button>
  <div>
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

