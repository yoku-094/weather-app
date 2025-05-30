<template>
  <WeatherTop />
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
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}
</style>
