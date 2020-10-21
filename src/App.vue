<template>
  <div id="app">
    <info />
    <mainapp />
    <foot />
  </div>
</template>

<script>
import info from "./components/info.vue";
import mainapp from "./components/mainapp.vue";
import foot from "./components/foot.vue";

export default {
  name: "App",
  components: {
    info,
    mainapp,
    foot,
  },
};
</script>
