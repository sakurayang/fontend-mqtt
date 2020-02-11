<template>
  <div id="index">
    <Mqtt />
  </div>
</template>

<script>
import Mqtt from "@/components/Mqtt.vue";

export default {
  name: "Home",
  components: { Mqtt }
};
</script>

<style scoped>
#index {
  min-width: 100%;
  min-height: 100%;
  margin: 0;
  padding: 0;
  height: 100vh;
}
</style>
