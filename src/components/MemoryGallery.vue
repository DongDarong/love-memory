<script setup>
defineProps({
  cards: {
    type: Array,
    required: true,
  },
});
</script>

<template>
  <div class="mb-5 flex items-end justify-between gap-4 px-1 sm:mb-6">
    <div>
      <p class="text-[0.68rem] font-semibold uppercase tracking-[0.4em] text-slate-400 sm:text-xs">Gallery</p>
      <h2 class="mt-2 text-2xl font-semibold text-slate-800 sm:text-3xl">Captured chapters</h2>
    </div>
    <p class="hidden max-w-xs text-right text-sm leading-6 text-slate-500 md:block">
      Sized to stay readable on mobile while keeping the scrapbook feeling on larger screens.
    </p>
  </div>

  <div class="memory-grid px-1 pb-20 sm:pb-24">
    <article
      v-for="card in cards"
      :key="card.image"
      class="memory-card fade-in-up hover-lift relative cursor-pointer rounded-[1.25rem] bg-white p-3 pb-5 shadow-[0_18px_40px_rgba(148,163,184,0.18)] transition duration-500 sm:pb-7"
      :class="card.rotateClass"
      :style="{ animationDelay: card.delay }"
    >
      <div class="tape" :style="card.tapeStyle" />
      <div class="memory-image-wrap overflow-hidden rounded-2xl bg-gray-100">
        <img :src="card.image" class="h-full w-full object-cover transition-transform duration-700 hover:scale-110" />
      </div>
      <div v-if="card.caption" class="font-script mt-4 text-center text-[1.9rem] text-gray-600 sm:text-[2rem]">{{ card.caption }}</div>
    </article>
  </div>

  <div class="fade-in-up pb-10 text-center sm:pb-14" style="animation-delay: 1.5s">
    <p class="font-script text-2xl text-rose-400 sm:text-3xl">Made with love</p>
  </div>
</template>
