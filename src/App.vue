<template>
  <div id="app">
    <Header v-if="showHeader" />
    <router-view />
    <Footer v-if="showFooter" />
  </div>
</template>

<script>
import Header from './components/header/Header.vue';
import Footer from './components/Footer/Footer.vue'; 
import slid from './components/Slider/slid.vue';

export default {
  name: 'App',
  components: {
    Header,
    Footer,
    slid,
  },
  data() {
    return {
      showHeader: true,
      showFooter: true,
    };
  },

  created() {
    this.showHeader = this.$route.path !== '/reg';
    this.showFooter = this.$route.path !== '/reg';
  },
};
</script>

<style>
body {
  margin: 0;
}


</style>