<template>
  <div id="app">
   
    <ServiceRequestTracker />
  </div>
</template>

<script>
import ServiceRequestTracker from './components/ServiceRequestTracker.vue'

export default {
  name: 'app',
  components: {
    ServiceRequestTracker
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
