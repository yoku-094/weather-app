<template>
  <!-- 画面ちらつきの応急処置 -->
  <div class="loading-screen" v-if="condition == null"></div>

  <div class="main" v-else>
    <div class="select-city">
      <div class="select-city-items">
        <select v-model="selectCity" class="form-select">
          <option v-for="city in cities" :key="city.name" :value="city.name">
            {{ city.text }}
          </option>
        </select>
        <button @click="getCurrentLocationWeather" class="btn btn-primary">
          現在地の天気
        </button>
      </div>
    </div>
    <div class="weather-info-area">
      <div class="weather-info-now">
        <div class="weather-info-1">
          <div class="prefectures">{{ displayName }}</div>
          <div class="cities">{{ description }}</div>
        </div>
        <div class="weather-icon">
          <img :src="getWeatherIcon(condition)" alt="Weather Icon" />
        </div>
        <div class="weather-info-2">
          <p>
            <span class="weather-temp">{{ Math.round(temp) }} ℃</span>
          </p>
          <p>
            <span class="weather-pressure"
              >{{ atmospheric_pressure }} &ensp;hPa</span
            >
          </p>
        </div>
      </div>
      <div class="weekly-weather-info">
        <table class="table text-center">
          <thead>
            <tr>
              <th scope="col">日付</th>
              <th v-for="hour in hours" :key="hour" scope="col">
                {{ hour }}時
              </th>
            </tr>
          </thead>
          <tbody class="table-group-divider">
            <tr v-for="(day, index) in forecastData" :key="index">
              <td v-html="formatDate(day[0].date)"></td>
              <td v-for="(hourData, hourIndex) in day" :key="hourIndex">
                <div class="weekly-weather-icon">
                  <img
                    :src="getWeatherIcon(hourData.condition)"
                    alt="Weather Icon"
                  />
                </div>
                <div class="weekly-weather-description">
                  {{ hourData.description }}
                </div>
                <div class="weekly-weather-temp">
                  {{ Math.round(hourData.temp) }} °C
                </div>
                <div class="weekly-weather-pressure">
                  {{ hourData.pressure }} hPa
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
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
      temp: null,
      atmospheric_pressure: null,
      hours: [],
      forecastData: [],
      suppressWatch: false,
      isCurrentLocation: false,
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
        .then((res) => {
          this.city = res.data.name;
          this.description = res.data.weather[0].description;
          this.condition = res.data.weather[0].main;
          this.temp = res.data.main.temp;
          this.atmospheric_pressure = res.data.main.pressure;
          // 未来の天気を取得
          this.getForecast({
            q: this.selectCity,
          });

          this.isCurrentLocation = false;
          // セッション管理用
          sessionStorage.isCurrentLocation = this.isCurrentLocation;
        })
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
        this.temp = data.main.temp;
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

          this.isCurrentLocation = true;
          // セッション管理用
          sessionStorage.isCurrentLocation = this.isCurrentLocation;
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
              condition: item.weather[0].main,
              description: item.weather[0].description,
              rain: item.rain?.["3h"] || 0,
              wind: item.wind,
            });
          }
        });
        // 日付でソートして配列に変換
        const forecastArray = Object.keys(grouped)
          .sort()
          .map((date) => {
            // 各日0-21時でソート
            return grouped[date].sort((a, b) => a.hour - b.hour);
          })
          // 0-21時までのデータが揃っているもののみ
          .filter((dayArray) => dayArray.length === this.hours.length);
        this.forecastData = forecastArray;
      } catch (err) {
        console.error("天気取得エラー", err);
        alert("週間天気を取得できませんでした");
      }
    },
    getWeatherIcon(condition) {
      const weatherIconMap = {
        Clear: require("@/assets/logo_Sunny.png"),
        Clouds: require("@/assets/logo_Cloud.png"),
        Rain: require("@/assets/logo_Rain.png"),
        Snow: require("@/assets/logo_Snow.png"),
      };
      return weatherIconMap[condition] || require("@/assets/logo_Other.png");
    },
    formatDate(dateString) {
      const date = new Date(dateString);
      const month = date.getMonth() + 1;
      const day = date.getDate();
      const weekDays = ["日", "月", "火", "水", "木", "金", "土"];
      const week = weekDays[date.getDay()];
      return `${month}/${day}<br>（${week}）`;
    },
  },
  mounted() {
    if (sessionStorage.isCurrentLocation === "true") {
      // 現在地ボタン
      this.getCurrentLocationWeather();
    } else if (sessionStorage.selectCity) {
      // プルダウン選択
      this.selectCity = sessionStorage.selectCity;
      this.getWeather();
      this.getDisplayName();
      this.isCurrentLocation = false;
    } else {
      // どちらもなければ初期表示
      this.getWeather();
      this.getDisplayName();
      this.isCurrentLocation = false;
    }
    sessionStorage.isCurrentLocation = this.isCurrentLocation;
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
table td {
  align-content: center;
  padding: 10px 20px;
}

.main {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.select-city {
  display: flex;
  justify-content: center;
  padding: 20px 0px;
}
.select-city-items {
  display: flex;
  flex-direction: column;
  text-align-last: center;
  width: 140px;
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

.weather-info-area {
  display: flex;
  flex: 1;
  justify-content: space-evenly;
  padding: 20px;
  column-gap: 20px;
}

.weather-info-2,
.weekly-weather-info {
  font-weight: bold;
}

.weather-info-now {
  padding: 0 20px;
  height: fit-content;
  align-self: center;
}
.weather-temp {
  color: #ea85ac;
}
.weather-pressure {
  color: #858585;
}

.weekly-weather-info {
  display: flex;
}
.weekly-weather-icon img {
  width: 30px;
}
.weekly-weather-description {
  font-size: 12px;
  color: #858585;
}
.weekly-weather-temp {
  font-size: 15px;
  color: #ea85ac;
}
.weekly-weather-pressure {
  font-size: 15px;
  color: #858585;
}

@media (max-width: 600px) {
  table th,
  table td {
    padding: 6px;
    font-size: 12px;
  }
  .weather-info-area {
    flex-direction: column;
    align-items: center;
    padding: 10px;
  }

  .weather-info-now,
  .weekly-weather-info {
    padding: 10px 0;
    width: 100%;
  }

  .weekly-weather-info {
    justify-content: center;
    overflow-x: auto;
  }

  .weather-icon img {
    width: 200px;
  }

  .weekly-weather-description {
    height: 36px;
    align-content: center;
  }
  .weekly-weather-temp,
  .weekly-weather-pressure {
    font-size: 12px;
  }
}
</style>
