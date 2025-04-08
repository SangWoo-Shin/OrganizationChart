<template>
  <div class="min-h-screen bg-gray-300 text-gray-800">
    <AppHeader />
    <CsvUploader />
  </div>
</template>

<script >
import CsvUploader from './components/CsvUploader.vue'
import AppHeader from './components/Header.vue'

export default {
  name: 'App',
  components: {
    CsvUploader,
    AppHeader,
  }
}
</script>

<style src="../public/style/tree.css"></style> 