<template>
  <div id="app">
    <creditForm />
  </div>
</template>

<script>
import creditForm from './components/creditForm.vue'

export default {
  name: 'app',
  components: {
    creditForm
  }
}
</script>

<style>
  @import './assets/css/style.css';
</style>
