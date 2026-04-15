<script setup>
defineProps({
  hearts: {
    type: Array,
    required: true,
  },
});
</script>

<template>
  <div class="hearts-bg">
    <div
      v-for="heart in hearts"
      :key="heart.id"
      class="heart"
      :style="{
        left: heart.left,
        animationDuration: heart.duration,
        animationDelay: heart.delay,
      }"
    />
  </div>
</template>
