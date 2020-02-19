<template>
  <div id="app">
    <SpaceDatetimePicker></SpaceDateTimePicker>
    <SpaceDaterangePicker></SpaceDaterangePicker>
  </div>
</template>

<script>
import SpaceDatetimePicker from '@/components/SpaceDatetimePicker.vue';
import SpaceDaterangePicker from '@/components/SpaceDaterangePicker.vue';

export default {
  name: 'App',
  components: {
    SpaceDatetimePicker,
    SpaceDaterangePicker,
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
