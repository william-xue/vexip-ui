<template>
  <Viewer ref="viewer" width="1000" height="500">
    <img :src="images[active]" />
  </Viewer>
  <div class="image-list">
    <img
      v-for="(img, index) in images"
      :key="index"
      :src="img"
      :class="{
        'is-active': active === index
      }"
      @click="active = index"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'

import type { Viewer } from 'vexip-ui'

const viewer = ref<InstanceType<typeof Viewer>>()
const active = ref(0)
const images = [
  'https://www.vexipui.com/picture-1.jpg',
  'https://www.vexipui.com/picture-2.jpg',
  'https://www.vexipui.com/picture-3.jpg',
  'https://www.vexipui.com/picture-4.jpg',
  'https://www.vexipui.com/picture-5.jpg',
]

watch(active, reset)

function reset() {
  viewer.value?.handleReset()
}
</script>

<style scoped>
.image-list {
  display: flex;
  justify-content: center;
  width: 1000px;
  margin-top: 10px;
}

.image-list img {
  width: 100px;
  height: 60px;
  margin-inline-end: 10px;
  user-select: none;
  opacity: 60%;
}

.image-list img:last-child {
  margin-inline-end: 0;
}

.image-list img.is-active {
  opacity: 100%;
}
</style>
