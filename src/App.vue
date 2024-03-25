<script>
import axios from 'axios';
import { RouterLink, RouterView } from 'vue-router'
import './style.css'

export default{
  components:{
    RouterView,
  },
  mounted(){
    window.onkeyup = (e) =>{
        if(e.key == 'a' && e.ctrlKey && e.altKey){
        this.$router.push('/admin')
      }
    }
  }
} 



</script>

<template>
  <RouterView />


</template>

<style scoped></style>
