<template>
  <div id="app" align=center>
    <Grafico/>
  </div>
</template>

<script>
import Grafico from './components/Grafico.vue';

export default {
  name: 'App',
  components: {
    Grafico
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  height: 1000px;
  width: 846px;
  align-content: center;
  align-self: center;
}
</style>
