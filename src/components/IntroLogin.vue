<script setup>
const password = defineModel('password', {
  type: String,
  required: true,
});

defineProps({
  content: {
    type: Object,
    required: true,
  },
  hasError: {
    type: Boolean,
    default: false,
  },
  isShaking: {
    type: Boolean,
    default: false,
  },
  introClasses: {
    type: Object,
    required: true,
  },
});

const emit = defineEmits(['submit']);

function handleKeypress(event) {
  if (event.key === 'Enter') {
    emit('submit');
  }
}
</script>

<template>
  <section
    class="fixed inset-0 z-50 flex flex-col items-center justify-center bg-[radial-gradient(circle_at_top,_rgba(255,255,255,0.95),_rgba(255,241,246,0.92)_48%,_rgba(252,231,243,0.82)_100%)] px-4 py-6 transition-all duration-1000 sm:p-6"
    :class="introClasses"
  >
    <div
      class="glass-panel relative w-full max-w-[23rem] overflow-hidden rounded-[2rem] px-5 py-8 text-center transition-all duration-300 sm:max-w-md sm:px-8 md:px-14 md:py-12"
      :class="{ shake: isShaking }"
    >
      <div class="absolute -top-12 -right-8 h-28 w-28 rounded-full bg-pink-300/70 blur-2xl sm:h-32 sm:w-32" />
      <div class="absolute -bottom-12 -left-8 h-28 w-28 rounded-full bg-orange-200/70 blur-2xl sm:h-32 sm:w-32" />

        <div class="relative z-10">
        <p class="mb-3 text-[0.65rem] font-semibold uppercase tracking-[0.45em] text-rose-400 sm:text-xs">{{ content.eyebrow }}</p>
        <h1 class="font-script mb-3 text-[3rem] leading-none text-rose-500 sm:text-6xl">{{ content.title }}</h1>
        <p class="mx-auto mb-7 max-w-xs text-sm leading-6 text-slate-500 sm:mb-8 sm:text-base">
          {{ content.description }}
        </p>

        <div class="relative group">
          <input
            v-model="password"
            type="password"
            :placeholder="content.placeholder"
            class="w-full rounded-2xl border border-white/70 bg-white/80 px-4 py-3.5 text-center text-base text-slate-700 shadow-[inset_0_1px_1px_rgba(255,255,255,0.85),0_8px_24px_rgba(244,114,182,0.08)] transition-all placeholder:text-slate-400 focus:border-rose-300 focus:bg-white focus:outline-none sm:text-lg"
            :class="{ 'border-rose-500': hasError }"
            @keypress="handleKeypress"
          />
        </div>

        <p
          v-if="hasError"
          class="mt-4 inline-block rounded-full bg-rose-100 px-3 py-1 text-sm font-bold text-rose-500"
        >
          {{ content.errorMessage }}
        </p>

        <button
          class="group relative mt-5 inline-flex w-full items-center justify-center overflow-hidden rounded-2xl bg-gradient-to-r from-rose-400 via-pink-400 to-orange-300 px-8 py-3.5 font-medium tracking-tighter text-white shadow-[0_14px_35px_rgba(244,114,182,0.35)] transition-all duration-300 hover:-translate-y-1"
          @click="$emit('submit')"
        >
          <span class="absolute h-0 w-0 rounded-full bg-white opacity-10 transition-all duration-500 ease-out group-hover:h-80 group-hover:w-80" />
          <span class="relative flex items-center justify-center gap-2 font-bold tracking-wide">
            {{ content.buttonLabel }}
            <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14 5l7 7m0 0-7 7m7-7H3" />
            </svg>
          </span>
        </button>

        <div class="mt-6 grid grid-cols-3 gap-2 text-left sm:gap-3">
          <div
            v-for="card in content.summaryCards"
            :key="card.label"
            class="rounded-2xl bg-white/55 px-3 py-3"
          >
            <p class="text-[0.62rem] font-semibold uppercase tracking-[0.25em] text-slate-400">{{ card.label }}</p>
            <p class="mt-1 text-lg font-bold text-slate-700">{{ card.value }}</p>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>
