<template>

  <IndexView ></IndexView>
  
</template>

<script>
import IndexView from './job/index.vue';


export default {
  name: 'MainView',
  props: {
    global: Object
  },
  components:{
    IndexView
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
