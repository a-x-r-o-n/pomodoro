<template>
  <div :class="[
    'min-h-screen flex items-center justify-center font-sans p-4',
    sessionType === 'Focus'
      ? 'bg-gradient-to-tr from-black via-zinc-900 to-red-900 text-white'
      : sessionType === 'Short Break'
      ? 'bg-gradient-to-tr from-black via-zinc-900 to-emerald-600 text-white'
      : 'bg-gradient-to-tr from-black via-zinc-900 to-indigo-900 text-white'
  ]">
    <div class="space-y-10">
      <div class="text-center">
        <h1 class="text-4xl font-bold">Pomodoro <span :class="    sessionType === 'Focus'
      ? 'text-red-500'
      : sessionType === 'Short Break'
      ? 'text-emerald-600'
      : 'text-indigo-500'">Timer</span></h1>
      </div>
            <!-- Session Switcher -->
      <div class="flex justify-center gap-4 text-sm font-medium">
        <button @click="manualSetSession('Focus')" :class="sessionType === 'Focus' ? 'bg-red-600' : 'bg-zinc-700 hover:bg-zinc-600'" class="px-4 py-2 rounded-xl transition">
          Focus
        </button>
        <button @click="manualSetSession('Short Break')" :class="sessionType === 'Short Break' ? 'bg-emerald-600' : 'bg-zinc-700 hover:bg-zinc-600'" class="px-4 py-2 rounded-xl transition">
          Short Break
        </button>
        <button @click="manualSetSession('Long Break')" :class="sessionType === 'Long Break' ? 'bg-indigo-600' : 'bg-zinc-700 hover:bg-zinc-600'" class="px-4 py-2 rounded-xl transition">
          Long Break
        </button>
      </div>
      <!-- Timer Card -->
      <div class="flex justify-center">
        <div class="bg-zinc-950 rounded-3xl p-8 shadow-2xl w-80">
          <div class="flex justify-center mb-6">
            <svg class="w-32 h-32" viewBox="0 0 100 100">
              <circle cx="50" cy="50" r="45" stroke="#333" stroke-width="10" fill="none" />
              <circle
                cx="50"
                cy="50"
                r="45"
                :stroke="sessionType === 'Focus' ? '#ff4500' : sessionType === 'Short Break' ? '#22c55e' : '#6366f1'"
                stroke-width="10"
                fill="none"
                stroke-linecap="round"
                :stroke-dasharray="circumference"
                :stroke-dashoffset="dashOffset"
                transform="rotate(-90 50 50)"
              />
              <text x="50%" y="54%" dominant-baseline="middle" text-anchor="middle" class="fill-white text-xl font-mono">
                {{ formattedTime }}
              </text>
            </svg>
          </div>

          <div class="text-center text-sm text-zinc-400 mb-4 uppercase">{{ sessionType }}</div>

          <div class="flex justify-around">
            <button
              @click="toggleTimer"
              class="px-6 py-2 bg-zinc-800 rounded-xl hover:bg-zinc-700 transition">
              {{ isRunning ? 'Pause' : 'Start' }}
            </button>
            <button @click="resetTimer" class="px-6 py-2 bg-zinc-700 rounded-xl hover:bg-zinc-600 transition">
              Reset
            </button>
          </div>
        </div>
      </div>


      <!-- Customization Modal Trigger -->
      <div class="flex justify-center">
        <button @click="showSettings = true" class="bg-zinc-800 px-6 py-2 rounded-xl hover:bg-zinc-700 transition text-sm">Customize Time</button>
      </div>

      <div class="text-center text-zinc-300 text-xs">
        <p>Focus: {{ focusDuration / 60 }} mins • Short Break: {{ shortBreakDuration / 60 }} mins • Long Break: {{ longBreakDuration / 60 }} mins</p>
      </div>
    </div>

    

    <!-- Settings Modal -->
    <div v-if="showSettings" class="fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center z-50">
      <div class="bg-zinc-800 text-white rounded-xl p-6 w-[90%] max-w-md">
        <h2 class="text-xl font-semibold mb-4">Customize Session Durations</h2>
        <div class="space-y-4">
          <label class="block">Focus (mins):
            <input type="number" v-model.number="customFocus" min="1" class="w-full mt-1 px-3 py-2 rounded bg-zinc-900 text-white border border-zinc-600" />
          </label>
          <label class="block">Short Break (mins):
            <input type="number" v-model.number="customShort" min="1" class="w-full mt-1 px-3 py-2 rounded bg-zinc-900 text-white border border-zinc-600" />
          </label>
          <label class="block">Long Break (mins):
            <input type="number" v-model.number="customLong" min="1" class="w-full mt-1 px-3 py-2 rounded bg-zinc-900 text-white border border-zinc-600" />
          </label>
        </div>
        <div class="mt-6 flex justify-end gap-2">
          <button @click="showSettings = false" class="px-4 py-2 bg-zinc-700 hover:bg-zinc-600 rounded">Cancel</button>
          <button @click="applyCustomDurations" class="px-4 py-2 bg-red-600 hover:bg-red-500 rounded">Apply</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue';

let focusDuration = ref(25 * 60);
let shortBreakDuration = ref(5 * 60);
let longBreakDuration = ref(15 * 60);

const sessionType = ref('Focus');
const timeLeft = ref(focusDuration.value);
const isRunning = ref(false);
const interval = ref(null);
const sessionsCompleted = ref(0);

const customFocus = ref(focusDuration.value / 60);
const customShort = ref(shortBreakDuration.value / 60);
const customLong = ref(longBreakDuration.value / 60);

const showSettings = ref(false);

const bell = new Audio('https://www.soundjay.com/buttons/sounds/beep-07.mp3');

const toggleTimer = () => {
  if (isRunning.value) {
    clearInterval(interval.value);
    isRunning.value = false;
  } else {
    isRunning.value = true;
    interval.value = setInterval(() => {
      timeLeft.value--;
      if (timeLeft.value <= 0) {
        clearInterval(interval.value);
        isRunning.value = false;
        bell.play();
        switchSession();
      }
    }, 1000);
  }
};

const resetTimer = () => {
  clearInterval(interval.value);
  isRunning.value = false;
  setSession(sessionType.value);
};

const setSession = (type) => {
  sessionType.value = type;
  timeLeft.value =
    type === 'Focus' ? focusDuration.value : type === 'Short Break' ? shortBreakDuration.value : longBreakDuration.value;
  localStorage.setItem('pomodoroType', type);
  localStorage.setItem('pomodoroTimeLeft', timeLeft.value);
};

const manualSetSession = (type) => {
  clearInterval(interval.value);
  isRunning.value = false;
  setSession(type);
};

const switchSession = () => {
  if (sessionType.value === 'Focus') {
    sessionsCompleted.value++;
    if (sessionsCompleted.value % 4 === 0) {
      setSession('Long Break');
    } else {
      setSession('Short Break');
    }
  } else {
    setSession('Focus');
  }
};

const applyCustomDurations = () => {
  focusDuration.value = customFocus.value * 60;
  shortBreakDuration.value = customShort.value * 60;
  longBreakDuration.value = customLong.value * 60;
  setSession(sessionType.value);
  showSettings.value = false;
};

const formattedTime = computed(() => {
  const min = String(Math.floor(timeLeft.value / 60)).padStart(2, '0');
  const sec = String(timeLeft.value % 60).padStart(2, '0');
  return `${min}:${sec}`;
});

const circumference = 2 * Math.PI * 45;
const dashOffset = computed(() => (1 - timeLeft.value / getSessionDuration()) * circumference);

const getSessionDuration = () => {
  return sessionType.value === 'Focus'
    ? focusDuration.value
    : sessionType.value === 'Short Break'
    ? shortBreakDuration.value
    : longBreakDuration.value;
};

if (localStorage.getItem('pomodoroType')) {
  sessionType.value = localStorage.getItem('pomodoroType');
}
if (localStorage.getItem('pomodoroTimeLeft')) {
  timeLeft.value = parseInt(localStorage.getItem('pomodoroTimeLeft'));
}

watch(timeLeft, (newVal) => {
  localStorage.setItem('pomodoroTimeLeft', newVal);
});
</script>

<style scoped>
svg text {
  font-size: 14px;
}
</style>
