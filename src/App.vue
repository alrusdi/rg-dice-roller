<script setup lang="ts">
import Menu from './components/Menu.vue'
import Main from './components/Main.vue'

import { ref } from 'vue'

const currentPage = ref("menu")
const playersCount = ref(4);

</script>

<template>
  <div v-if="currentPage === 'menu'">
    <Menu v-on:change-players-count="c => {playersCount=c; currentPage='main'}" />
  </div>
  <div v-if="currentPage === 'main'">
    <Main :playersCount="playersCount" v-on:new-game="currentPage='menu'" />
  </div>
</template>

<style scoped>

</style>
