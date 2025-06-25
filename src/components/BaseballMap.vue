<template>
  <div class="map-wrapper">
    <img src="@/assets/japan-map.png" class="map-image" />

    <div
      v-for="stadium in stadiums"
      :key="stadium.name"
      class="marker"
      :style="{ top: stadium.top + 'px', left: stadium.left + 'px' }"
      @click="selectStadium(stadium)"
    >
      ⚾
    </div>

    <p v-if="selectedStadium">選んだ球場：{{ selectedStadium.name }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      stadiums: [
        { name: "東京ドーム", top: 250, left: 310 },
        { name: "甲子園", top: 380, left: 200 },
        // ここに他の球場も追加可能
      ],
      selectedStadium: null
    };
  },
  methods: {
    selectStadium(stadium) {
      this.selectedStadium = stadium;
    }
  }
};
</script>

<style scoped>
.map-wrapper {
  position: relative;
  width: 600px; /* 地図の幅に合わせて調整 */
}

.map-image {
  width: 100%;
}

.marker {
  position: absolute;
  cursor: pointer;
  font-size: 20px;
  transform: translate(-50%, -50%);
}
</style>
