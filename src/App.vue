<template>
  <div>
    <h1>⚾ 球場お天気アプリ</h1>
    <BaseballMap />
  </div>
</template>

<script>
import BaseballMap from './components/BaseballMap.vue';

export default {
  components: {
    BaseballMap
  }
};
</script>

<style>
/* 必要なら見た目のスタイルをここに書いてOK */
</style>
