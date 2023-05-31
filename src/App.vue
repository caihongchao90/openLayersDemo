<template>
  <!-- <olmap></olmap> -->
  <DrawMap></DrawMap>
</template>
<script>   
import olmap from './components/olMap.vue'
import DrawMap from './components/DrawMap.vue'
export default {
  name:"app",
  components:{
    olmap,
    DrawMap
}
}
</script>

<style>
</style>

