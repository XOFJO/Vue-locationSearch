<script setup>
import { RouterView } from 'vue-router';
import Header from './components/Header.vue';
import Footer from './components/Footer.vue';
</script>

<template>
  <div class="flex flex-col min-h-screen w-screen">
    <Header />
    <RouterView />
    <Footer />
  </div>
</template>

<style scoped></style>
