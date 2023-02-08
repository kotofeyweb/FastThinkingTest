<script setup lang="ts">
import TheGallery from "./components/TheGallery/index.vue";
</script>

<template>
  <main>
    <h1>Simple Gallery</h1>
    <TheGallery />
  </main>
</template>

<style lang="scss">
body {
  font: 12px/16px serif;
  color: #000;
  overflow-y: scroll;
}

main {
  max-width: 1366px;
  padding: 15px;
  margin: 30px auto;
}
</style>
