<template>
  <div class="map-wrapper">
    <!-- 日本地図の画像 -->
    <img src="@/assets/japan-map.png" class="map-image" />

    <!-- 各球場をマーカーで表示し、クリックできるようにする -->
    <div
      v-for="stadium in stadiums"
      :key="stadium.name"
      class="marker"
      :style="{ top: stadium.top + 'px', left: stadium.left + 'px' }"
      @click="selectStadium(stadium)"
    >
      ⚾
    </div>

    <!-- 球場をクリックしたときに名前を表示 -->
    <p v-if="selectedStadium">選んだ球場：{{ selectedStadium.name }}</p>

    <!-- 天気情報があれば表示する -->
    <div v-if="weatherList.length">
      <h3>📅 {{ selectedStadium.name }}の天気</h3>
      <ul>
        <li v-for="weather in weatherList" :key="weather.dt">
          {{ weather.dt_txt }}：
          {{ weather.weather[0].description }}、
          気温：{{ weather.main.temp }}℃、
          降水量：{{ getRain(weather) }} mm、
          <!-- judgeWeather() を呼んで観戦日和判定を表示 -->
          <strong>{{ judgeWeather(weather) }}</strong>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 球場リスト：緯度・経度・座標・屋内外の設定
      stadiums: [
        {
          name: "東京ドーム",
          lat: 35.7056,
          lon: 139.7519,
          top: 250,
          left: 310,
          isOutdoor: false // 屋内球場
        },
        {
          name: "甲子園",
          lat: 34.721,
          lon: 135.3612,
          top: 380,
          left: 200,
          isOutdoor: true // 野外球場
        }
      ],
      selectedStadium: null,     // 選択された球場
      weatherList: []            // 天気予報データ
    };
  },

  methods: {
    // 球場をクリックしたときに天気データを取得する
    async selectStadium(stadium) {
      console.log("クリックされた球場：", stadium); // 確認用
      this.selectedStadium = stadium;

      const apiKey = "5e9b429c65a28ebae947e241ccfd6ee7";
      const url = `https://api.openweathermap.org/data/2.5/forecast?lat=${stadium.lat}&lon=${stadium.lon}&appid=${apiKey}&units=metric&lang=ja`;

      try {
        const response = await fetch(url);
        const data = await response.json();
        console.log(data.list); // 確認用
        // 最初の3日分のデータだけセット
        this.weatherList = data.list.slice(0, 8 * 3);
      } catch (error) {
        console.error("天気データの取得に失敗:", error);
      }
    },

    // 降水量（mm）を安全に取り出すメソッド
    getRain(weather) {
      // rain['3h']がなければ0を返す
      return weather.rain && weather.rain['3h'] ? weather.rain['3h'] : 0;
    },

    // 天気データから観戦日和かどうか判定するメソッド
    judgeWeather(weather) {
      // 3時間ごとの降水量(mm)
      const rain = weather.rain && weather.rain['3h'] ? weather.rain['3h'] : 0;
      // 気温（摂氏）
      const temp = weather.main.temp;
      // 湿度（%）
      const humidity = weather.main.humidity;
      // 選択された球場の屋内外判定（selectedStadiumがnullの可能性も考慮）
      const isOutdoor = this.selectedStadium ? this.selectedStadium.isOutdoor : false;

      if (!isOutdoor) {
        // 屋内球場の場合は雨だけ判定
        return rain >= 1 ? "☂️ 道中、傘が必要です" : "😊 傘はいりません";
      } else {
        // 野外球場の場合は詳細判定
        if (rain >= 1) return "☔ 雨で観戦つらい";
        if (rain > 0) return "🌧️ 小雨ですが観戦可能";
        if (temp >= 30 && humidity >= 70) return "🥵 熱中症注意！湿度も高いです";
        return "☀️ 観戦日和です！";
      }
    }
  }
};
</script>

<style scoped>
/* マップ全体のスタイル */
.map-wrapper {
  position: relative;
  width: 600px; /* 地図の横幅に合わせて調整 */
}

/* 地図画像を親要素にフィットさせる */
.map-image {
  width: 100%;
}

/* 球場マーカー（⚾）のスタイル */
.marker {
  position: absolute;
  cursor: pointer;
  font-size: 20px;
  transform: translate(-50%, -50%); /* 中央に表示させるため */
}
</style>
