<template>
  <div id="app">
    <UploadMarkdown />
  </div>
</template>

<script>
import UploadMarkdown from './components/UploadMarkdown.vue'

export default {
  name: 'App',
  components: {
    UploadMarkdown
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
