<template>
  <div class="grid grid-cols-12 gap-4 h-screen">
    <Neo4jViz class="col-span-9 h-full" />
    <Chat class="col-span-3 h-full border-l border-gray-600/40" />
  </div>
</template>

<script setup>
import Neo4jViz from "./components/Neo4jViz.vue";
import Chat from "./components/Chat.vue";
</script>
