<template>
  <q-layout view="hHh lpR fFf">
    <q-page-container class="main-page">
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent, ref } from "vue";

export default defineComponent({
  name: "MainLayout",

  components: {},

  setup() {
    return {};
  },
});
</script>

<style>
.main-page {
  display: flex;
  justify-content: center;
  align-items: center; /* 垂直方向居中 */
  flex-direction: column; /* 保持子元素垂直排列 */
  width: 100%; /* 確保容器占滿寬度 */
  /* min-height: 100vh; 確保容器占滿高度 */
  /* padding: 20px; 根據需要調整內邊距 */
  box-sizing: border-box; /* 包括填充在內計算元素的寬度和高度 */
}
</style>
