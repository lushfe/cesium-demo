<template>
  <div id="app">
    <ZarlGisComp />
  </div>
</template>

<script>
import ZarlGisComp from "@/components/ZarlGis/ZarlGisComp.vue";
export default {
  name: "App",
  components: {
    ZarlGisComp,
  },
};
</script>

<style lang="scss">
#app {
  height: 100vh;
  width: 100vw;
}
</style>
