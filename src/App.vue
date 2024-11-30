<script setup>
import Header from './components/Header.vue'
import Footer from './components/Footer.vue'
import { RouterView } from "vue-router"
</script>

<template>
    <Header></Header>
  <RouterView v-slot="{ Component }">
    <transition name="slide-right" :duration="200">
      <component :is="Component" />
    </transition>
  </RouterView>
  <Footer></Footer>
</template>
