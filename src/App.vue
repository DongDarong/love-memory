<script setup>
import confetti from 'canvas-confetti';
import { computed, onBeforeUnmount, onMounted, ref } from 'vue';
import BackgroundHearts from './components/BackgroundHearts.vue';
import IntroLogin from './components/IntroLogin.vue';
import MemoryGallery from './components/MemoryGallery.vue';
import MemoryHero from './components/MemoryHero.vue';

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

const introClasses = computed(() => ({
  hidden: isIntroHidden.value,
  'opacity-0': isUnlocked.value,
  'opacity-100': !isUnlocked.value,
}));

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

    <BackgroundHearts :hearts="hearts" />

    <audio ref="audioRef" loop>
      <source :src="asset('song.mp3')" type="audio/mpeg" />
    </audio>

    <IntroLogin
      v-model:password="password"
      :has-error="hasError"
      :is-shaking="isShaking"
      :intro-classes="introClasses"
      @submit="checkLogin"
    />

    <section
      v-show="isUnlocked"
      class="relative z-10 mx-auto w-full max-w-7xl px-4 pb-0 pt-5 sm:px-6 sm:pt-7 md:px-8"
    >
      <MemoryHero
        :profile-images="profileImages"
        :stats="stats"
        :elapsed="elapsed"
      />

      <MemoryGallery :cards="galleryCards" />
    </section>
  </main>
</template>
