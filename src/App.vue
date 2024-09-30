<template>
  <div style="text-align: center; margin-top: 50px">
    <h1>Exam Timer Thing</h1>
    I am doing the <input v-model="paperName" @focus="isInputFocused = true" @blur="isInputFocused = false"
      placeholder="Skibidology Sigma College 2022" class="input"> paper
    <div>
      Start my question at number <input v-model.number="startQuestionNumber" type="number" class="input">
    </div>
    <div>
      End my exam at <input v-model.number="endExamTime" type="number" class="input"> minutes
    </div>
    <div v-if="!isRunning">Paused</div>
    <div style="font-size: 3rem">{{ formatTime(time, false) }} minutes</div>
    P: Start/Stop | ENTER: Split | J: Show/Hide Timer | Hold R for 5s: Reset<br>
    Timer is {{ isVisible ? 'visible' : 'hidden' }}
    <ul>
      <li>
        <div class="card">
          <b>Reading Time: {{ formatTime(readingTime) }}</b>
        </div>
      </li>
      <li v-for="(split, index) in splits" :key="index" v-if="!isReadingTime">
        <div class="card">
          Question {{ startQuestionNumber + index }}: {{ isVisible ? formatTime(split) : "Timer hidden" }}
        </div>
      </li>
      <li v-if="!isReadingTime">
        <div class="card background">
          <b>Question {{ startQuestionNumber + splits.length }} : {{ isVisible ? formatTime(currentSplit) :
            "Timer hidden" }}</b>
        </div>
      </li>
    </ul>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, watch } from 'vue';

const time = ref<number>(0); // Total running time excluding reading time
const isRunning = ref<boolean>(false); // Timer state
const readingTime = ref<number>(0); // Reading time
const splits = ref<number[]>([]); // Stores split times
const currentSplit = ref<number>(0); // Time for the current split
const isVisible = ref<boolean>(false); // Timer visibility state, initially false
const paperName = ref<string>(''); // Paper name
const isInputFocused = ref<boolean>(false); // Input focus state
const startQuestionNumber = ref<number>(1); // Start question number
const endExamTime = ref<number>(0); // End exam time in minutes
let interval: number | null = null; // To store interval reference
let isReadingTime = ref<boolean>(true); // State to check if it's reading time
let resetTimeout: number | null = null; // To store reset timeout reference

const saveState = () => {
  const state = {
    time: time.value,
    readingTime: readingTime.value,
    splits: splits.value,
    currentSplit: currentSplit.value,
    isReadingTime: isReadingTime.value,
    paperName: paperName.value, // Save paper name
    startQuestionNumber: startQuestionNumber.value, // Save start question number
    endExamTime: endExamTime.value, // Save end exam time
  };
  localStorage.setItem('timerState', JSON.stringify(state));
};

const loadState = () => {
  const state = localStorage.getItem('timerState');
  if (state) {
    const parsedState = JSON.parse(state);
    time.value = parsedState.time;
    readingTime.value = parsedState.readingTime;
    splits.value = parsedState.splits;
    currentSplit.value = parsedState.currentSplit;
    isReadingTime.value = parsedState.isReadingTime;
    paperName.value = parsedState.paperName; // Load paper name
    startQuestionNumber.value = parsedState.startQuestionNumber; // Load start question number
    endExamTime.value = parsedState.endExamTime; // Load end exam time
  }
};

const startStop = () => {
  if (isRunning.value) {
    if (interval) clearInterval(interval);
  } else {
    interval = window.setInterval(() => {
      if (isReadingTime.value) {
        readingTime.value += 10;
      } else {
        time.value += 10;
        currentSplit.value += 10;
      }
      saveState();
      if (endExamTime.value > 0 && time.value >= endExamTime.value * 60000) {
        alert('Exam time is over!');
        startStop();
      }
    }, 10);
  }
  isRunning.value = !isRunning.value;
};

const split = () => {
  if (isReadingTime.value) {
    isReadingTime.value = false;
  } else {
    // Add current split time to the list and reset the current split
    splits.value.push(currentSplit.value);
    currentSplit.value = 0;
  }
  saveState();
};

const formatTime = (time: number, showSandMs = true): string => {
  const minutes = Math.floor(time / 60000);
  const seconds = Math.floor((time % 60000) / 1000);
  const milliseconds = ((time % 60000) % 1000) / 10;

  return `${minutes}${showSandMs ? `:${seconds < 10 ? `0${seconds}` : seconds}.${milliseconds < 10 ? `0${milliseconds}` : milliseconds}` : ''}`;
};

const handleKeyPress = (event: KeyboardEvent) => {
  if (isInputFocused.value) return; // Do nothing if input is focused

  if (event.key === 'p') {
    startStop();
  } else if (event.key === 'Enter') {
    split();
  } else if (event.key === 'j' || event.key === 'J') {
    isVisible.value = !isVisible.value;
  } else if (event.key === 'r' || event.key === 'R') {
    if (!resetTimeout) {
      resetTimeout = window.setTimeout(() => {
        resetTimer();
        resetTimeout = null;
      }, 5000);
    }
  }
};

const handleKeyUp = (event: KeyboardEvent) => {
  if (event.key === 'r' || event.key === 'R') {
    if (resetTimeout) {
      clearTimeout(resetTimeout);
      resetTimeout = null;
    }
  }
};

const resetTimer = () => {
  time.value = 0;
  readingTime.value = 0;
  splits.value = [];
  currentSplit.value = 0;
  isReadingTime.value = true;
  paperName.value = ''; // Reset paper name
  startQuestionNumber.value = 1; // Reset start question number
  endExamTime.value = 0; // Reset end exam time
  saveState();
};

watch([paperName, startQuestionNumber, endExamTime], () => {
  saveState();
});

onMounted(() => {
  // Load state from localStorage
  loadState();
  // Add event listener for 'Space', 'Enter', 'J', and 'R' keys
  window.addEventListener('keydown', handleKeyPress);
  window.addEventListener('keyup', handleKeyUp);
});

onBeforeUnmount(() => {
  // Clean up event listener and interval
  window.removeEventListener('keydown', handleKeyPress);
  window.removeEventListener('keyup', handleKeyUp);
  if (interval) clearInterval(interval);
  if (resetTimeout) clearTimeout(resetTimeout);
});
</script>

<style scoped>
ul {
  list-style-type: none;
}
</style>
