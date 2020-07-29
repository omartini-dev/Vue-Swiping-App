<template>
  <div id="cardapp">
    <swipeable-cards />
  </div>
</template>

<script>
import SwipeableCards from './components/SwipeableCards.vue'

export default {
  name: 'cardapp',
  components: { SwipeableCards }
}
</script>

<style>
body {
  margin: 0;
  z-index: 0;
}
#cardapp {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  padding-top:10px;
}
</style>
