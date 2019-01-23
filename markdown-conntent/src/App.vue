<template>
  <div id="app" class= "row justify-content-center">
    <markdownHead/>
    <div class="w-100"></div>
    <markdown-body class="container-fluid" />
  </div>
</template>

<script>
import markdownHead from './components/markdownHead'
import markdownBody from './components/markdownBody/markdownBody'

export default {
  name: 'App',
  components: {
    markdownHead,
    markdownBody
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
