<template>
  <div id="app">
    <Home/>
  </div>
</template>

<script>
import Home from './components/Home.vue'

export default {
  name: 'App',
  components: {
    Home
  }
}
</script>

<style>
@font-face {
  font-family: "CFF";
  src: local("CFF"),
   url(../public/font/SBBWeb-Roman.ttf) format("truetype");
}

#app {
  font-family: CFF;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 15px;
  height: calc(100vh - 30px);
}
</style>
