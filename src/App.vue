<script setup>
import MainComponent from "./components/MainComponent.vue";
</script>

<template>
  <div class="container">
    <MainComponent />
  </div>
</template>

<style lang="scss" scoped>
@import "@/assets/style.scss";
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: $primary-color;
}
</style>
