<template>
  <div id="app">
    <div class="container">
      <div class="max-w-550 mx-auto">
        <h1 class="text-center text-white mb-3">My Todos</h1>
        <todoItem/>
      </div>
    </div>
  </div>
</template>

<script>
import todoItem from './components/todoItem.vue'

export default {
  name: 'App',
  components: {
    todoItem
  }
}
</script>

<style lang="scss">
@import "./assets/all.scss";

#app {
  font-family: Times New Roman, Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  min-height: 100vh;
  background: #d2c8c3;
  padding: 50px 0px;
}
.max-w-550{
  max-width: 550px;
}
</style>
