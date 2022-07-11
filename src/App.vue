<template>
  <div id="main" class="d-flex justify-content-center align-items-center flex-column">
    <SearchBar/>
    <SearchResult v-show="clicked"/>
    <div id="footer" class="footer py-3 text-center">
      <div class="container">
        <span class="text-muted">&copy; 2022 <a href="https://github.com/humatej">humatej</a></span><br>
        <span class="text-muted">Thank you <a href="https://www.pexels.com/photo/beautiful-clouds-cloudy-dramatic-209831/">Pixabay</a> for the use of your background image</span>
      </div>
    </div>
  </div>
</template>
<script lang="ts">
import { defineComponent } from 'vue'
import { ref } from 'vue'

import SearchBar from './components/SearchBar.vue'
import SearchResult from './components/SearchResult.vue'

export default defineComponent({
  components: {
    SearchBar,
    SearchResult
  },
  data(){
    const clicked = ref<boolean>(false)

    return { clicked }
  }
})
</script>
<style lang="scss">
  body{
    padding: 0;
    margin: 0;
    background-color: #fafafa;
  }
  #main{
    width: 100%;
    height: 100vh;
    background-image: url("../public/pexels-pixabay-209831.jpg");
  }
  #footer{
    position: fixed;
    bottom: 0;
    width: 100%;
  }
</style>
