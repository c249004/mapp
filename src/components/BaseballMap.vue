<template>
  <div class="map-wrapper">
    <!-- 日本地図の画像 -->
    <img src="@/assets/japan-map.png" class="map-image" />

    <!-- 各球場をマーカーで表示し、クリックできるようにする -->
   <div
  v-for="stadium in stadiums"
  :key="stadium.name"
  class="marker"
  :style="{ top: stadium.top + '%', left: stadium.left + '%' }"
  @click="selectStadium(stadium)"
>
  ⚾ <span class="label">{{ stadium.name }}</span>
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
    top: 60.0,     // 400 / 800
    left: 67.1,    // 410 / 772
    isOutdoor: false
  },
  {
    name: "甲子園",
    lat: 34.721,
    lon: 135.3612,
    top: 69.0,     // 400 / 800
    left: 42.0,    // 255 / 772
    isOutdoor: true
  },
  {
    name: "エスコンフィールドHOKKAIDO",
    lat: 43.7657,
    lon: 141.6935,
    top: 15.75,    // 110 / 800
    left: 70.7,    // 430 / 772
    isOutdoor: false
  },
  {
    name: "楽天モバイルパーク宮城",
    lat: 38.2602,
    lon: 140.9021,
    top: 45.5,     // 300 / 800
    left: 72.0,    // 440 / 772
    isOutdoor: true
  },
  {
    name: "ベルーナドーム",
    lat: 35.7684,
    lon: 139.4204,
    top: 62.75,    // 390 / 800
    left: 65.5,    // 390 / 772
    isOutdoor: true
  },
  {
    name: "ZOZOマリンスタジアム",
    lat: 35.6328,
    lon: 140.0347,
    top: 64.75,    // 390 / 800
    left: 70.4,    // 420 / 772
    isOutdoor: true
  },
  {
    name: "京セラドーム大阪",
    lat: 34.6693,
    lon: 135.4762,
    top: 65.5,     // 420 / 800
    left: 40.0,    // 255 / 772
    isOutdoor: false
  },
  {
    name: "みずほPayPayドーム",
    lat: 33.595,
    lon: 130.3626,
    top: 70.375,   // 435 / 800
    left: 12.3,    // 95 / 772
    isOutdoor: false
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
  width: 772px;
  height: 800px; /* 地図画像と同じに固定 */
  margin-bottom: 150px; /* 天気情報とマーカーが重ならないよう余白追加 */
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
  transform: translateY(-50%); /* ← 横方向のズレをなくす */
  white-space: nowrap;
  display: flex;
  align-items: center;
  gap: 4px;
}



/* 球場名ラベルの表示（⚾の右側に） */
.marker-label {
  position: absolute;
  left: 1.2em;  /* ボールの右に表示 */
  top: 50%;
  transform: translateY(-50%);
  background: white;
  padding: 2px 4px;
  border-radius: 4px;
  font-size: 14px;
  white-space: nowrap;
  box-shadow: 0 0 2px rgba(0,0,0,0.2);
}

</style>
