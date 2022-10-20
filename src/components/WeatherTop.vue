<template>
  <!-- 画面ちらつきの応急処置 -->
  <div class="loading-screen" v-if="condition == null"></div>

  <div class="main" v-else>
    <div class="select-city">
      <select v-model="selectCity">
        <option v-for="city in cities" :key="city.name" :value="city.name">
          {{ city.text }}
        </option>
      </select>
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
          "https://api.openweathermap.org/data/2.5/weather?appid=eb7c0edbd3e8755921136e3e1fbc239b&lang=ja&units=metric",
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
        .catch((error) => console.log(error));
    },
    // API取得の都市名に不正確なものがあるため、jsonファイルで設定し取得・表示する
    getDisplayName() {
      for (var list in this.cities) {
        if (this.cities[list].name == this.selectCity) {
          this.displayName = this.cities[list].display;
        }
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
