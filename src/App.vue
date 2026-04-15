<script setup>
import confetti from 'canvas-confetti';
import { computed, onBeforeUnmount, onMounted, ref, watch } from 'vue';
import BackgroundHearts from './components/BackgroundHearts.vue';
import IntroLogin from './components/IntroLogin.vue';
import MemoryGallery from './components/MemoryGallery.vue';
import MemoryHero from './components/MemoryHero.vue';
import siteContent from './mocks/site-content.json';

const currentLanguage = ref('en');
const languageOptions = [
  { value: 'en', label: 'EN' },
  { value: 'kh', label: 'KH' },
];

const assetModules = import.meta.glob('../image/*', {
  eager: true,
  import: 'default',
});

function asset(name) {
  return assetModules[`../image/${name}`];
}

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

const localizedContent = computed(() => siteContent[currentLanguage.value]);
const startDate = computed(() => new Date(localizedContent.value.relationship.anniversaryDate));
const passwordValue = computed(() => localizedContent.value.auth.password);
const heroProfiles = computed(() => localizedContent.value.profiles.map((profile) => ({
  ...profile,
  image: asset(profile.image),
})));
const galleryCards = computed(() => localizedContent.value.gallery.cards.map((card) => ({
  ...card,
  image: asset(card.image),
  tapeStyle: card.tapeStyle ?? {},
})));
const stats = computed(() => [
  { label: localizedContent.value.relationship.stats[0], value: elapsed.value.years },
  { label: localizedContent.value.relationship.stats[1], value: elapsed.value.months },
  { label: localizedContent.value.relationship.stats[2], value: elapsed.value.days },
]);

const introClasses = computed(() => ({
  hidden: isIntroHidden.value,
  'opacity-0': isUnlocked.value,
  'opacity-100': !isUnlocked.value,
}));

const pageLanguage = computed(() => (currentLanguage.value === 'kh' ? 'km' : 'en'));

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
  const baseDate = startDate.value;
  let years = now.getFullYear() - baseDate.getFullYear();
  let months = now.getMonth() - baseDate.getMonth();
  let days = now.getDate() - baseDate.getDate();

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
  if (password.value === passwordValue.value) {
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

watch(pageLanguage, (value) => {
  document.documentElement.lang = value;
}, { immediate: true });

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
  <main :lang="pageLanguage" class="love-shell relative min-h-screen overflow-hidden">
    <div class="ambient ambient-one" />
    <div class="ambient ambient-two" />
    <div class="ambient ambient-three" />

    <div class="fixed right-4 bottom-4 z-[60] sm:right-8 sm:bottom-8">
      <div class="glass-panel inline-flex items-center gap-1 rounded-full p-1 shadow-[0_10px_40px_rgba(225,29,72,0.15)]">
        <button
          v-for="option in languageOptions"
          :key="option.value"
          type="button"
          class="rounded-full px-4 py-2 text-xs font-bold tracking-[0.22em] transition-all"
          :class="currentLanguage === option.value ? 'bg-rose-500 text-white shadow-md' : 'text-slate-500 hover:text-rose-400'"
          @click="currentLanguage = option.value"
        >
          {{ option.label }}
        </button>
      </div>
    </div>

    <BackgroundHearts :hearts="hearts" />

    <audio ref="audioRef" loop>
      <source :src="asset(siteContent.shared.audio.file)" type="audio/mpeg" />
    </audio>

    <IntroLogin
      v-model:password="password"
      :content="localizedContent.auth"
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
        :content="localizedContent.relationship"
        :profiles="heroProfiles"
        :stats="stats"
        :elapsed="elapsed"
      />

      <MemoryGallery :content="localizedContent.gallery" :cards="galleryCards" />
    </section>
  </main>
</template>
