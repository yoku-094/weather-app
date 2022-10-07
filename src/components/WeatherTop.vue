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
      <h1>{{ city }}</h1>
      <h2>{{ description }}</h2>
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
      <p>hight: {{ max_temp }}</p>
      <p>low: {{ min_temp }}</p>
      <p>pressure: {{ atmospheric_pressure }}</p>
    </div>
    <div class="info-source">
      <p>
        source:&ensp;
        <a href="https://openweathermap.org/" target="_blank"
          >Open Weather API</a
        >
      </p>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "WeatherTop",
  data() {
    return {
      selectCity: "tokyo",
      cities: [
        { name: "sapporo", text: "札幌" },
        { name: "tokyo", text: "東京" },
        { name: "nagoya", text: "名古屋" },
        { name: "osaka", text: "大阪" },
        { name: "fukuoka", text: "福岡" },
      ],
      city: null,
      description: null,
      condition: null,
      max_temp: null,
      min_temp: null,
      atmospheric_pressure: null,
    };
  },
  mounted() {
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
  watch: {
    selectCity(newSelectCity) {
      this.selectCity = newSelectCity;

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
</style>
