<template>
  <img alt="Vue logo" src="./assets/logo.png">
  <Timer />
  <footer>
    <a href="https://github.com/ToshihitoKon/panepon-timer">github</a>
  </footer>
</template>

<script>
import Timer from './components/Timer.vue'

export default {
  name: 'App',
  components: {
    Timer
  }
}
</script>

<style>
#app {
  text-align: center;
}
</style>
