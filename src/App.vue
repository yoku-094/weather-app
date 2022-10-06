<template>
  <WeatherTop />
  <footer>
    <p>
      This information is displayed using the
      <a href="https://openweathermap.org/" target="_blank">Open Weather API</a>
      .
    </p>
  </footer>
</template>

<script>
import WeatherTop from "./components/WeatherTop.vue";

export default {
  name: "App",
  components: {
    WeatherTop,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
