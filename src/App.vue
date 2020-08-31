<template>
  <div id="app">
<!--    <clock/>-->
    <Clock02/>
    <canvas-nest/>
    <mu/>
  </div>
</template>

<script>
  import Mu from './components/common/mu/Mu'
  import Clock from "./components/common/Clock";
  import Clock02 from "./components/common/Clock02";
  import CanvasNest from "./components/common/CanvasNest";
  export default {
    components: {
      Mu,
      Clock,
      Clock02,
      CanvasNest
    }
  }
</script>
<style scoped>
  #app{
    height: 100vh;
  }
</style>
