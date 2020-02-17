<template>
  <div id="app">
    <SpaceDatetimePicker></SpaceDateTimePicker>
  </div>
</template>

<script>
import SpaceDatetimePicker from '@/components/SpaceDatetimePicker.vue';

export default {
  name: 'App',
  components: {
    SpaceDatetimePicker,
  },
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
