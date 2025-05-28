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
    <div class="weather-info-now">
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
    <div class="weather-info-week">
      <table class="table table-bordered text-center">
        <thead>
          <tr>
            <th>日付</th>
            <th v-for="hour in hours" :key="hour">{{ hour }}時</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(day, index) in forecastData" :key="index">
            <td>{{ day[0].date }}</td>
            <!-- dayは配列なので日付は最初の要素から -->
            <td v-for="(hourData, hourIndex) in day" :key="hourIndex">
              <!-- <div class="weather-icon">{{ hourData.icon }}</div> -->
              <div>{{ hourData.temp }} °C</div>
              <div>{{ hourData.pressure }} hPa</div>
            </td>
          </tr>
        </tbody>
      </table>
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
      hours: [],
      forecastData: [],
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
            (this.atmospheric_pressure = res.data.main.pressure),
            // 未来の天気を取得
            this.getForecast({
              q: this.selectCity,
            })
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
        // 未来の天気予報を取得
        await this.getForecast({
          lat: lat,
          lon: lon,
        });

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
    async getForecast({ q, lat, lon }) {
      try {
        const params = {
          appid: apiKey,
          lang: "ja",
          units: "metric",
        };

        if (lat && lon) {
          // 現在地の天気ボタン押下時
          params.lat = lat;
          params.lon = lon;
        } else if (q) {
          // プルダウン選択時
          params.q = q;
        } else {
          alert("都市名または緯度経度を指定してください");
          return;
        }

        const weeklyWeather = await axios.get(
          "https://api.openweathermap.org/data/2.5/forecast",
          {
            params,
          }
        );

        this.hours = [0, 6, 9, 12, 15, 18, 21];
        const grouped = {};
        // 必要な時間のデータを抽出
        weeklyWeather.data.list.forEach((item) => {
          const jst = new Date(
            new Date(item.dt * 1000).toLocaleString("ja-JP", {
              timeZone: "Asia/Tokyo",
            })
          );
          const hour = jst.getHours();
          const date = jst.toISOString().split("T")[0];

          // 対象の時間のみ抽出
          if (this.hours.includes(hour)) {
            if (!grouped[date]) grouped[date] = [];

            grouped[date].push({
              date,
              hour,
              temp: item.main.temp,
              pressure: item.main.pressure,
              weather: item.weather[0].main,
              description: item.weather[0].description,
              // icon: item.weather[0].icon,
              rain: item.rain?.["3h"] || 0,
              wind: item.wind,
            });
          }
        });
        // 日付でソートして配列に変換
        const forecastArray = Object.keys(grouped)
          .sort()
          .map((date) => {
            // 各日の配列もhourでソートして返す
            return grouped[date].sort((a, b) => a.hour - b.hour);
          });
        this.forecastData = forecastArray;
      } catch (err) {
        console.error("天気取得エラー", err);
        alert("週間天気を取得できませんでした");
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
