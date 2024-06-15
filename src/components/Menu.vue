<script setup lang="ts">
import { ref } from 'vue'

const playersCount = ref(0)
</script>

<template>
  <div class="menu mt-5">
    <h3>How many players?</h3>

    <div class="mt-5">
      <div class="mb-2">
        <button class="btn btn-primary w-25" v-on:click="$emit('changePlayersCount', 2)">2</button>
      </div>
      <div class="mb-2">
        <button class="btn btn-primary w-25" v-on:click="$emit('changePlayersCount', 3)">3</button>
      </div>
      <div class="mb-2">
        <button class="btn btn-primary w-25" v-on:click="$emit('changePlayersCount', 4)">4</button>
      </div>
    </div>
  </div>
</template>

<style scoped>

</style>
