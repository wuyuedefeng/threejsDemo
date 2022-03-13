<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup
import BuildingsRenderer from './components/BuildingsRenderer'
</script>

<template>
  <BuildingsRenderer></BuildingsRenderer>
</template>

<style>
html, body , #app {
  width: 100%;
  height: 100%;
}

body {
  margin: 0;
  padding: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
