<template>
  <div id="app">
    <NutritionFactsLabel />
  </div>
</template>

<script>
import NutritionFactsLabel from './components/NutritionFactsLabel.vue';

export default {
  name: 'App',
  components: {
    NutritionFactsLabel
  }
}
</script>

<style>
#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin-top: 60px;
}
</style>
