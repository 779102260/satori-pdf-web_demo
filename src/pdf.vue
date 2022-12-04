<!-- PDF 模板 -->
<script setup lang="ts">
import { computed } from "vue";

const props = defineProps({
  title: {
    required: true,
    type: String
  },
  website:  {
    required: true,
    type: String
  }
})

const data = computed(() => props.title + props.website + new Date().toLocaleDateString())

</script>

<template>
  <div class="box" style="display: flex;flex-direction: column;justify-content: center;align-items: center;border: 1px solid #ccc;">
    <h1 class="title" style="color: red;">
      {{ title }}
    </h1>
    <div class="address">
      {{ website }}
    </div>
    <p tw="text-2xl">{{data}}</p>
  </div>
</template>
<style>
/* NOTE v-staori只支持内联样式和tw，否则需要单独处理 */
.box {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.title {
  color: red;
}
</style>