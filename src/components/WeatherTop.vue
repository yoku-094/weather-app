<template>
  <!-- 画面ちらつきの応急処置 -->
  <div class="loading-screen" v-if="condition == null"></div>

  <div class="main" v-else>
    <div class="select-city">
      <div class="select-city-items">
        <select v-model="selectCity">
          <option v-for="city in cities" :key="city.name" :value="city.name">
            {{ city.text }}
          </option>
        </select>
        <button @click="getCurrentLocationWeather">現在地の天気</button>
      </div>
    </div>
    <div class="weather-info-1">
      <div class="prefectures">{{ displayName }}</div>
      <div class="cities">{{ description }}</div>
    </div>
    <div class="weather-icon">
      <p v-if="condition == 'Clear'">
        <img alt="logo_sunny" src="../assets/logo_Sunny.png" />
      </p>
      <p v-else-if="condition == 'Clouds'">
        <img alt="logo_cloud" src="../assets/logo_Cloud.png" />
      </p>
      <p v-else-if="condition == 'Rain'">
        <img alt="logo_rain" src="../assets/logo_Rain.png" />
      </p>
      <p v-else>
        <img alt="logo_other" src="../assets/logo_Other.png" />
      </p>
    </div>
    <div class="weather-info-2">
      <p>
        <span class="max-temp">{{ Math.round(max_temp) }} ℃</span>
      </p>
      <p>
        <span class="min-temp">{{ Math.round(min_temp) }} ℃</span>
      </p>
      <p>
        <span class="pressure">{{ atmospheric_pressure }} &ensp;hPa</span>
      </p>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import cityList from "../assets/data/city-list.json";

const apiKey = process.env.VUE_APP_WEATHER_API_KEY;

export default {
  name: "WeatherTop",
  data() {
    return {
      selectCity: "shinjuku",
      cities: cityList,
      city: null,
      displayName: null,
      description: null,
      condition: null,
      max_temp: null,
      min_temp: null,
      atmospheric_pressure: null,
    };
  },
  methods: {
    getWeather() {
      axios
        .get(
          `https://api.openweathermap.org/data/2.5/weather?appid=${apiKey}&lang=ja&units=metric`,
          {
            params: {
              q: this.selectCity,
            },
          }
        )
        .then(
          (res) => (
            (this.city = res.data.name),
            (this.description = res.data.weather[0].description),
            (this.condition = res.data.weather[0].main),
            (this.max_temp = res.data.main.temp_max),
            (this.min_temp = res.data.main.temp_min),
            (this.atmospheric_pressure = res.data.main.pressure)
          )
        )
        .catch((error) => {
          console.log(error);
          alert("現在地の天気を取得できませんでした");
        });
    },
    // API取得の都市名に不正確なものがあるため、jsonファイルで設定し取得・表示する
    getDisplayName() {
      for (var list in this.cities) {
        if (this.cities[list].name == this.selectCity) {
          this.displayName = this.cities[list].display;
        }
      }
    },
    async getCurrentLocationWeather() {
      try {
        const position = await new Promise((resolve, reject) => {
          navigator.geolocation.getCurrentPosition(resolve, reject);
        });

        const lat = position.coords.latitude;
        const lon = position.coords.longitude;

        // 市区町村名（displayName）を reverse geocoding で取得
        const geoRes = await axios.get(
          "https://api.openweathermap.org/geo/1.0/reverse",
          {
            params: {
              lat,
              lon,
              limit: 1,
              appid: apiKey,
            },
          }
        );
        const location = geoRes.data[0];
        this.displayName = location.local_names.ja;

        // 天気データを緯度経度で取得
        const weatherRes = await axios.get(
          "https://api.openweathermap.org/data/2.5/weather",
          {
            params: {
              lat,
              lon,
              appid: apiKey,
              lang: "ja",
              units: "metric",
            },
          }
        );
        const data = weatherRes.data;
        this.city = data.name; // 参考に保持
        this.description = data.weather[0].description;
        this.condition = data.weather[0].main;
        this.max_temp = data.main.temp_max;
        this.min_temp = data.main.temp_min;
        this.atmospheric_pressure = data.main.pressure;

        // 国土地理院 API を呼び出す（都道府県コード取得）
        const gsiRes = await axios.get(
          "https://mreversegeocoder.gsi.go.jp/reverse-geocoder/LonLatToAddress",
          {
            params: {
              lat,
              lon,
            },
          }
        );
        const muniCd = gsiRes.data.results.muniCd;
        // 都道府県コード（先頭の二桁のみ）の取得
        const prefCode = muniCd.slice(0, 2);

        // cityList から該当する都道府県コードを持つ都市を探す
        const match = this.cities.find((city) => city.prefCode === prefCode);

        if (match) {
          this.selectCity = match.name;
          // プルダウン選択による都道府県選択の抑制用
          this.suppressWatch = true;
        }
      } catch (err) {
        console.error("天気取得エラー", err);
        alert("現在地の天気を取得できませんでした");
      }
    },
  },
  mounted() {
    if (sessionStorage.selectCity) {
      this.selectCity = sessionStorage.selectCity;
    }

    this.getWeather();
    this.getDisplayName();
  },
  watch: {
    selectCity(newSelectCity) {
      if (this.suppressWatch) {
        // 現在地ボタン押下時のプルダウン選択はスキップ
        this.suppressWatch = false;
        return;
      }

      this.selectCity = newSelectCity;
      sessionStorage.selectCity = newSelectCity;

      this.getWeather();
      this.getDisplayName();
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

.select-city {
  display: flex;
  justify-content: center;
}
.select-city-items {
  display: flex;
  flex-direction: column;
  text-align-last: center;
  width: 100px;
  row-gap: 14px;
}
.prefectures {
  font-size: 2.5rem;
  font-weight: bold;
  padding: 20px 0px 25px;
}
.cities {
  font-size: 1.5rem;
  font-weight: bold;
}

.weather-info-2 {
  font-weight: bold;
}
.max-temp {
  color: #ea85ac;
}
.min-temp {
  color: #85beea;
}
.pressure {
  color: #858585;
}
</style>
