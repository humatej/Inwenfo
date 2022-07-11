<template>
  <div id="main" class="d-flex justify-content-center align-items-center flex-column overflow-hidden">
    <SearchBar/>
    <div id="footer" class="footer py-3 text-center">
      <div class="container">
        <div>
          <span class="text-light">&copy; 2022 Matej Hušla<br>
        full info <a href="https://github.com/humatej/Iwenfo">docs</a></span><br>
        </div>
      </div>
    </div>
  </div>
</template>
<script lang="ts">
import { defineComponent } from 'vue'
import { ref } from 'vue'

import SearchBar from './components/SearchBar.vue'

export default defineComponent({
  components: {
    SearchBar,
  },
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
    background-image: url("../public/adventure-5077752_1920.jpg");
    background-repeat:no-repeat;
    background-position:center;
  }
  #footer{
    position: fixed;
    bottom: 0;
    width: 100%;
    z-index: 0;
  }
  #footer a{
    font-weight: bold;
    color: rgb(231, 211, 24);
  }
</style>
