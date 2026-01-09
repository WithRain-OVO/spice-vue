<template>
  <div id="app">
    <HomePage v-if="currentPage === 'home'" />
    <SpiceViewer v-if="currentPage === 'spice'" />
  </div>
</template>

<script>
import HomePage from './components/HomePage.vue'
import SpiceViewer from './components/SpiceViewer.vue'

export default {
  name: 'App',
  components: {
    HomePage,
    SpiceViewer
  },
  data() {
    return {
      currentPage: 'home'
    }
  },
  mounted() {
    /**
     * 根据URL参数决定显示哪个页面
     */
    const urlParams = new URLSearchParams(window.location.search);
    const page = urlParams.get('page');
    if (page === 'spice') {
      this.currentPage = 'spice';
    } else {
      this.currentPage = 'home';
    }
  }
}
</script>

<style>
#app {
  width: 100%;
  height: 100vh;
  margin: 0;
  padding: 0;
}
</style>
