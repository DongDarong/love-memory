<script setup>
defineProps({
  content: {
    type: Object,
    required: true,
  },
  profiles: {
    type: Array,
    required: true,
  },
  stats: {
    type: Array,
    required: true,
  },
  elapsed: {
    type: Object,
    required: true,
  },
});
</script>

<template>
  <div class="fade-in-up grid gap-8 pb-12 lg:grid-cols-[minmax(0,1.15fr)_minmax(18rem,0.85fr)] lg:items-start" style="animation-delay: 0.1s">
    <div class="text-center lg:pt-6 lg:text-left">
      <div class="mb-6 flex items-end justify-center gap-4 sm:gap-5 lg:justify-start">
        <div class="group relative flex flex-col items-center">
          <div class="profile-ring" :class="profiles[0].ringClass" />
          <div class="relative h-20 w-20 overflow-hidden rounded-full border-[5px] border-white bg-gray-100 shadow-xl sm:h-24 sm:w-24 md:h-28 md:w-28">
            <img :src="profiles[0].image" :alt="profiles[0].alt" class="h-full w-full object-cover" />
          </div>
          <h1 class="font-script mt-3 text-xl text-gray-700 drop-shadow-sm transition-colors duration-300 sm:text-2xl md:text-3xl" :class="profiles[0].hoverClass">
            {{ profiles[0].name }}
          </h1>
        </div>

        <div class="heart-badge mb-9 flex h-14 w-14 items-center justify-center rounded-full text-rose-400 shadow-lg sm:h-16 sm:w-16">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-7 w-7 fill-current sm:h-8 sm:w-8" viewBox="0 0 20 20">
            <path fill-rule="evenodd" d="M3.172 5.172a4 4 0 0 1 5.656 0L10 6.343l1.172-1.171a4 4 0 1 1 5.656 5.656L10 17.657l-6.828-6.829a4 4 0 0 1 0-5.656z" clip-rule="evenodd" />
          </svg>
        </div>

        <div class="group relative flex flex-col items-center">
          <div class="profile-ring" :class="profiles[1].ringClass" />
          <div class="relative h-20 w-20 overflow-hidden rounded-full border-[5px] border-white bg-gray-100 shadow-xl sm:h-24 sm:w-24 md:h-28 md:w-28">
            <img :src="profiles[1].image" :alt="profiles[1].alt" class="h-full w-full object-cover" />
          </div>
          <h1 class="font-script mt-3 text-xl text-gray-700 drop-shadow-sm transition-colors duration-300 sm:text-2xl md:text-3xl" :class="profiles[1].hoverClass">
            {{ profiles[1].name }}
          </h1>
        </div>
      </div>

      <p class="mb-3 text-[0.68rem] font-semibold uppercase tracking-[0.45em] text-rose-400 sm:text-xs">{{ content.heroEyebrow }}</p>
      <h1 class="font-script mx-auto max-w-xl text-[3.3rem] leading-[0.95] text-slate-800 drop-shadow-sm sm:text-6xl md:text-7xl lg:mx-0 lg:max-w-2xl">
        {{ content.title }}
      </h1>
      <p class="mx-auto mt-4 max-w-xl text-sm leading-7 text-slate-500 sm:text-base lg:mx-0">
        {{ content.description }}
      </p>

      <div class="glass-panel mx-auto mt-6 inline-block max-w-xl rounded-[1.75rem] px-5 py-4 sm:px-6 lg:mx-0">
        <p class="font-serif text-base italic leading-7 text-gray-600 md:text-lg">
          "{{ content.quote }}"
        </p>
      </div>
    </div>

    <div class="hero-panel glass-panel rounded-[2rem] p-4 sm:p-5 lg:mt-2">
      <div class="mb-4 flex items-center justify-between gap-3">
        <div>
          <p class="text-[0.68rem] font-semibold uppercase tracking-[0.35em] text-rose-400 sm:text-xs">{{ content.sinceLabel }}</p>
          <h2 class="mt-2 text-xl font-semibold text-slate-800 sm:text-2xl">{{ content.sinceDisplay }}</h2>
        </div>
        <div class="rounded-2xl bg-white/70 px-3 py-2 text-right shadow-sm">
          <p class="text-[0.62rem] uppercase tracking-[0.25em] text-slate-400">{{ content.statusLabel }}</p>
          <p class="mt-1 text-sm font-semibold text-emerald-500">{{ content.statusValue }}</p>
        </div>
      </div>

      <div class="stat-grid">
        <div
          v-for="stat in stats"
          :key="stat.label"
          class="stat-card"
        >
          <span class="font-serif block text-[2.2rem] font-bold leading-none text-slate-800 sm:text-5xl">{{ stat.value }}</span>
          <span class="mt-2 text-[0.7rem] font-bold uppercase tracking-[0.28em] text-rose-400">{{ stat.label }}</span>
        </div>
      </div>

      <div class="glass-panel mt-4 grid grid-cols-3 gap-2 rounded-[1.5rem] px-3 py-3 text-center sm:mt-5 sm:gap-3 sm:px-4 sm:py-4">
        <div class="time-chip">
          <span class="time-chip__value">{{ elapsed.hours }}</span>
          <span class="time-chip__label">{{ content.timeLabels[0] }}</span>
        </div>
        <div class="time-chip">
          <span class="time-chip__value">{{ elapsed.minutes }}</span>
          <span class="time-chip__label">{{ content.timeLabels[1] }}</span>
        </div>
        <div class="time-chip">
          <span class="time-chip__value">{{ elapsed.seconds }}</span>
          <span class="time-chip__label">{{ content.timeLabels[2] }}</span>
        </div>
      </div>

      <div class="mt-4 rounded-[1.5rem] bg-white/70 px-4 py-4 text-left shadow-sm sm:mt-5">
        <p class="text-[0.68rem] font-semibold uppercase tracking-[0.3em] text-slate-400">{{ content.moodLabel }}</p>
        <p class="mt-2 text-sm leading-6 text-slate-500">
          {{ content.moodDescription }}
        </p>
      </div>
    </div>
  </div>
</template>
