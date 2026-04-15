<script setup>
import confetti from 'canvas-confetti';
import { computed, onBeforeUnmount, onMounted, ref } from 'vue';

const startDate = new Date(2025, 3, 9);
const passwordValue = '090325';

const assetModules = import.meta.glob('../image/*', {
  eager: true,
  import: 'default',
});

function asset(name) {
  return assetModules[`../image/${name}`];
}

const profileImages = {
  him: asset('20.jpg'),
  her: asset('16.jpg'),
};

const galleryCards = [
  { image: asset('21.jpg'), caption: 'First Date', rotateClass: '-rotate-2', delay: '0.4s', tapeStyle: {} },
  { image: asset('12.jpg'), caption: 'Sweet Moments', rotateClass: 'rotate-1', delay: '0.5s', tapeStyle: { transform: 'translateX(-50%) rotate(2deg)' } },
  { image: asset('13.jpg'), caption: 'Adventures', rotateClass: '-rotate-1', delay: '0.6s', tapeStyle: {} },
  { image: asset('4.jpg'), caption: 'Just Us', rotateClass: 'rotate-2', delay: '0.7s', tapeStyle: {} },
  { image: asset('23.jpg'), caption: '', rotateClass: '-rotate-2', delay: '0.8s', tapeStyle: {} },
  { image: asset('16.jpg'), caption: '', rotateClass: 'rotate-1', delay: '0.9s', tapeStyle: {} },
  { image: asset('22.jpg'), caption: '', rotateClass: 'rotate-2', delay: '1.0s', tapeStyle: {} },
  { image: asset('15.jpg'), caption: '', rotateClass: '-rotate-1', delay: '1.1s', tapeStyle: {} },
  { image: asset('14.jpg'), caption: '', rotateClass: 'rotate-1', delay: '1.2s', tapeStyle: {} },
  { image: asset('18.jpg'), caption: 'Forever', rotateClass: '-rotate-2', delay: '1.3s', tapeStyle: {} },
];

const hearts = ref([]);
const isUnlocked = ref(false);
const isIntroHidden = ref(false);
const password = ref('');
const hasError = ref(false);
const isShaking = ref(false);
const audioRef = ref(null);

const elapsed = ref({
  years: 0,
  months: 0,
  days: 0,
  hours: '00',
  minutes: '00',
  seconds: '00',
});

const stats = computed(() => [
  { label: 'Years', value: elapsed.value.years },
  { label: 'Months', value: elapsed.value.months },
  { label: 'Days', value: elapsed.value.days },
]);

let timerId;
let introTimeoutId;
let shakeTimeoutId;

function createHearts() {
  hearts.value = Array.from({ length: 15 }, (_, index) => ({
    id: index,
    left: `${Math.random() * 100}vw`,
    duration: `${Math.random() * 5 + 10}s`,
    delay: `${Math.random() * 10}s`,
  }));
}

function updateTimer() {
  const now = new Date();
  let years = now.getFullYear() - startDate.getFullYear();
  let months = now.getMonth() - startDate.getMonth();
  let days = now.getDate() - startDate.getDate();

  if (days < 0) {
    months -= 1;
    const prevMonth = new Date(now.getFullYear(), now.getMonth(), 0);
    days += prevMonth.getDate();
  }

  if (months < 0) {
    years -= 1;
    months += 12;
  }

  elapsed.value = {
    years,
    months,
    days,
    hours: now.getHours().toString().padStart(2, '0'),
    minutes: now.getMinutes().toString().padStart(2, '0'),
    seconds: now.getSeconds().toString().padStart(2, '0'),
  };
}

function clearErrorState() {
  hasError.value = false;
  isShaking.value = false;
  if (shakeTimeoutId) {
    clearTimeout(shakeTimeoutId);
    shakeTimeoutId = undefined;
  }
}

function fireSweetConfetti() {
  const duration = 3000;
  const end = Date.now() + duration;

  function frame() {
    confetti({
      particleCount: 6,
      angle: 60,
      spread: 55,
      origin: { x: 0 },
      colors: ['#ffb6c1', '#ff69b4', '#ff1493'],
      shapes: ['circle'],
    });

    confetti({
      particleCount: 6,
      angle: 120,
      spread: 55,
      origin: { x: 1 },
      colors: ['#ffb6c1', '#ff69b4', '#ff1493'],
      shapes: ['circle'],
    });

    if (Date.now() < end) {
      requestAnimationFrame(frame);
    }
  }

  frame();
}

async function revealSurprise() {
  isUnlocked.value = true;
  clearErrorState();

  introTimeoutId = window.setTimeout(() => {
    isIntroHidden.value = true;
  }, 1000);

  if (audioRef.value) {
    audioRef.value.volume = 0.5;
    try {
      await audioRef.value.play();
    } catch (error) {
      console.error(error);
    }
  }

  fireSweetConfetti();
}

function checkLogin() {
  if (password.value === passwordValue) {
    revealSurprise();
    return;
  }

  password.value = '';
  hasError.value = true;
  isShaking.value = true;

  if (shakeTimeoutId) {
    clearTimeout(shakeTimeoutId);
  }

  shakeTimeoutId = window.setTimeout(() => {
    isShaking.value = false;
  }, 500);
}

function handleEnter(event) {
  if (event.key === 'Enter') {
    checkLogin();
  }
}

const introClasses = computed(() => ({
  hidden: isIntroHidden.value,
  'opacity-0': isUnlocked.value,
  'opacity-100': !isUnlocked.value,
}));

onMounted(() => {
  createHearts();
  updateTimer();
  timerId = window.setInterval(updateTimer, 1000);
});

onBeforeUnmount(() => {
  if (timerId) {
    clearInterval(timerId);
  }

  if (introTimeoutId) {
    clearTimeout(introTimeoutId);
  }

  if (shakeTimeoutId) {
    clearTimeout(shakeTimeoutId);
  }
});
</script>

<template>
  <main class="love-shell relative min-h-screen overflow-hidden">
    <div class="ambient ambient-one" />
    <div class="ambient ambient-two" />
    <div class="ambient ambient-three" />

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

    <audio ref="audioRef" loop>
      <source :src="asset('song.mp3')" type="audio/mpeg" />
    </audio>

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
          <p class="mb-3 text-[0.65rem] font-semibold uppercase tracking-[0.45em] text-rose-400 sm:text-xs">Private Memory Archive</p>
          <h1 class="font-script mb-3 text-[3rem] leading-none text-rose-500 sm:text-6xl">Love Story</h1>
          <p class="mx-auto mb-7 max-w-xs text-sm leading-6 text-slate-500 sm:mb-8 sm:text-base">
            A small space for your photos, music, and the time you have been together.
          </p>

          <div class="relative group">
            <input
              v-model="password"
              type="password"
              placeholder="Enter Password"
              class="w-full rounded-2xl border border-white/70 bg-white/80 px-4 py-3.5 text-center text-base text-slate-700 shadow-[inset_0_1px_1px_rgba(255,255,255,0.85),0_8px_24px_rgba(244,114,182,0.08)] transition-all placeholder:text-slate-400 focus:border-rose-300 focus:bg-white focus:outline-none sm:text-lg"
              :class="{ 'border-rose-500': hasError }"
              @keypress="handleEnter"
            />
          </div>

          <p
            v-if="hasError"
            class="mt-4 inline-block rounded-full bg-rose-100 px-3 py-1 text-sm font-bold text-rose-500"
          >
            Wrong password! Try again.
          </p>

          <button
            class="group relative mt-5 inline-flex w-full items-center justify-center overflow-hidden rounded-2xl bg-gradient-to-r from-rose-400 via-pink-400 to-orange-300 px-8 py-3.5 font-medium tracking-tighter text-white shadow-[0_14px_35px_rgba(244,114,182,0.35)] transition-all duration-300 hover:-translate-y-1"
            @click="checkLogin"
          >
            <span class="absolute h-0 w-0 rounded-full bg-white opacity-10 transition-all duration-500 ease-out group-hover:h-80 group-hover:w-80" />
            <span class="relative flex items-center justify-center gap-2 font-bold tracking-wide">
              LOGIN
              <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14 5l7 7m0 0-7 7m7-7H3" />
              </svg>
            </span>
          </button>

          <div class="mt-6 grid grid-cols-3 gap-2 text-left sm:gap-3">
            <div class="rounded-2xl bg-white/55 px-3 py-3">
              <p class="text-[0.62rem] font-semibold uppercase tracking-[0.25em] text-slate-400">Photos</p>
              <p class="mt-1 text-lg font-bold text-slate-700">10+</p>
            </div>
            <div class="rounded-2xl bg-white/55 px-3 py-3">
              <p class="text-[0.62rem] font-semibold uppercase tracking-[0.25em] text-slate-400">Music</p>
              <p class="mt-1 text-lg font-bold text-slate-700">1</p>
            </div>
            <div class="rounded-2xl bg-white/55 px-3 py-3">
              <p class="text-[0.62rem] font-semibold uppercase tracking-[0.25em] text-slate-400">Access</p>
              <p class="mt-1 text-lg font-bold text-slate-700">Locked</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section
      v-show="isUnlocked"
      class="relative z-10 mx-auto w-full max-w-7xl px-4 pb-0 pt-5 sm:px-6 sm:pt-7 md:px-8"
    >
      <div class="fade-in-up grid gap-8 pb-12 lg:grid-cols-[minmax(0,1.15fr)_minmax(18rem,0.85fr)] lg:items-start" style="animation-delay: 0.1s">
        <div class="text-center lg:pt-6 lg:text-left">
          <div class="mb-6 flex items-end justify-center gap-4 sm:gap-5 lg:justify-start">
            <div class="group relative flex flex-col items-center">
              <div class="profile-ring bg-gradient-to-r from-blue-300 to-indigo-400" />
              <div class="relative h-20 w-20 overflow-hidden rounded-full border-[5px] border-white bg-gray-100 shadow-xl sm:h-24 sm:w-24 md:h-28 md:w-28">
                <img :src="profileImages.him" alt="Him" class="h-full w-full object-cover" />
              </div>
              <h1 class="font-script mt-3 text-xl text-gray-700 drop-shadow-sm transition-colors duration-300 group-hover:text-indigo-500 sm:text-2xl md:text-3xl">
                Darong
              </h1>
            </div>

            <div class="heart-badge mb-9 flex h-14 w-14 items-center justify-center rounded-full text-rose-400 shadow-lg sm:h-16 sm:w-16">
              <svg xmlns="http://www.w3.org/2000/svg" class="h-7 w-7 fill-current sm:h-8 sm:w-8" viewBox="0 0 20 20">
                <path fill-rule="evenodd" d="M3.172 5.172a4 4 0 0 1 5.656 0L10 6.343l1.172-1.171a4 4 0 1 1 5.656 5.656L10 17.657l-6.828-6.829a4 4 0 0 1 0-5.656z" clip-rule="evenodd" />
              </svg>
            </div>

            <div class="group relative flex flex-col items-center">
              <div class="profile-ring bg-gradient-to-r from-rose-300 to-pink-400" />
              <div class="relative h-20 w-20 overflow-hidden rounded-full border-[5px] border-white bg-gray-100 shadow-xl sm:h-24 sm:w-24 md:h-28 md:w-28">
                <img :src="profileImages.her" alt="Her" class="h-full w-full object-cover" />
              </div>
              <h1 class="font-script mt-3 text-xl text-gray-700 drop-shadow-sm transition-colors duration-300 group-hover:text-rose-500 sm:text-2xl md:text-3xl">
                Chiminh
              </h1>
            </div>
          </div>

          <p class="mb-3 text-[0.68rem] font-semibold uppercase tracking-[0.45em] text-rose-400 sm:text-xs">Forever Collection</p>
          <h1 class="font-script mx-auto max-w-xl text-[3.3rem] leading-[0.95] text-slate-800 drop-shadow-sm sm:text-6xl md:text-7xl lg:mx-0 lg:max-w-2xl">
            My Love
          </h1>
          <p class="mx-auto mt-4 max-w-xl text-sm leading-7 text-slate-500 sm:text-base lg:mx-0">
            A soft archive of your dates, adventures, and the little moments that made the story feel real.
          </p>

          <div class="glass-panel mx-auto mt-6 inline-block max-w-xl rounded-[1.75rem] px-5 py-4 sm:px-6 lg:mx-0">
            <p class="font-serif text-base italic leading-7 text-gray-600 md:text-lg">
              "Every love story is beautiful, but ours is my favorite."
            </p>
          </div>
        </div>

        <div class="hero-panel glass-panel rounded-[2rem] p-4 sm:p-5 lg:mt-2">
          <div class="mb-4 flex items-center justify-between gap-3">
            <div>
              <p class="text-[0.68rem] font-semibold uppercase tracking-[0.35em] text-rose-400 sm:text-xs">Together Since</p>
              <h2 class="mt-2 text-xl font-semibold text-slate-800 sm:text-2xl">09 April 2025</h2>
            </div>
            <div class="rounded-2xl bg-white/70 px-3 py-2 text-right shadow-sm">
              <p class="text-[0.62rem] uppercase tracking-[0.25em] text-slate-400">Status</p>
              <p class="mt-1 text-sm font-semibold text-emerald-500">Growing</p>
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
              <span class="time-chip__label">Hours</span>
            </div>
            <div class="time-chip">
              <span class="time-chip__value">{{ elapsed.minutes }}</span>
              <span class="time-chip__label">Minutes</span>
            </div>
            <div class="time-chip">
              <span class="time-chip__value">{{ elapsed.seconds }}</span>
              <span class="time-chip__label">Seconds</span>
            </div>
          </div>

          <div class="mt-4 rounded-[1.5rem] bg-white/70 px-4 py-4 text-left shadow-sm sm:mt-5">
            <p class="text-[0.68rem] font-semibold uppercase tracking-[0.3em] text-slate-400">Memory Mood</p>
            <p class="mt-2 text-sm leading-6 text-slate-500">
              Warm light, favorite song, and the kind of timeline that only keeps getting better.
            </p>
          </div>
        </div>
      </div>

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
          v-for="card in galleryCards"
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
    </section>
  </main>
</template>
