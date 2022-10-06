<template>
  <div class="hello">
    <h1>{{ city }}</h1>
    <h2>{{ description }}</h2>
    <p v-if="condition == 'Clear'">
      <img alt="logo_sunny" src="../assets/logo_Sunny.png" />
    </p>
    <p v-else-if="condition == 'Clouds'">
      <img alt="logo_cloud" src="../assets/logo_Cloud.png" />
    </p>
    <p v-else-if="condition == 'Rain'">
      <img alt="logo_rain" src="../assets/logo_Rain.png" />
    </p>
    <p v-else><img alt="logo_other" src="../assets/logo_Other.png" /></p>

    <p>hight: {{ max_temp }}</p>
    <p>low: {{ min_temp }}</p>
    <p>pressure: {{ atmospheric_pressure }}</p>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "WeatherTop",
  data() {
    return {
      city: "",
      description: "",
      condition: "",
      max_temp: "",
      min_temp: "",
      atmospheric_pressure: "",
    };
  },
  mounted() {
    axios
      .get(
        "https://api.openweathermap.org/data/2.5/weather?q=tokyo&appid=eb7c0edbd3e8755921136e3e1fbc239b&lang=ja&units=metric"
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
