<script>
async function loadFromDir(pattern, exclude) {
  const files = [];
  const parent = await window.showDirectoryPicker();
  if (!parent) {
    return;
  }
  for await (let handle of parent.values()) {
    if (!handle.name.match(exclude) && handle.name.match(pattern)) {
      files.push(handle);
    }
  }
  return files;
}

export default {
  data() {
    return {
      msg: "HEllo",
      files: [],
    };
  },
  methods: {
    async openDir() {
      this.files = await loadFromDir(/.+\.png/, /.+\.thumb\.png/);
    },
  },
};
</script>

<template>
  <button @click="openDir">Open Dir</button>
  <div>
    <div v-for="(file, i) in files" :key="i">
      {{ file.name }}
    </div>
  </div>
</template>

<style scoped>
a {
  color: #42b983;
}
</style>
