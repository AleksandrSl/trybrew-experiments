<script setup lang="ts">
import { computed, ref } from "vue";

let startTime: number;
let elapsedTime = ref(0);
let isRunning = false;
const stepIndex = ref(0)
const step = computed(() => steps[stepIndex.value])
const nextStep = computed(() => steps[stepIndex.value + 1])

const steps = [
  {
    time: 5_000,
    description: "In a spiral and intensely"
  },
  {
    time: 7_000,
    description: "From the center to the edges of the coffee tablet",
  },
  {
    time: 8_000,
    description: "To the center and back to spiral brewing",
  },
  {
    time: 8_000,
    description: "To the center",
  }
]

function startTimer() {
  isRunning = true;
  startTime = performance.now() - elapsedTime.value;
  requestAnimationFrame(updateTimer);
}

function stopTimer() {
  isRunning = false;
}

function resetTimer() {
  stopTimer();
  elapsedTime.value = 0;
}

function updateTimer(currentTime: number) {
  if (!isRunning) return;
  elapsedTime.value = currentTime - startTime;

  if (elapsedTime.value > step.value.time) {
    resetTimer()
    if (stepIndex.value + 1 >= steps.length) {
      console.debug("stopping brew", stepIndex.value, steps.length)
      stopBrew()
    } else {
      stepIndex.value = stepIndex.value + 1;
      startTimer()
    }
  }

  requestAnimationFrame(updateTimer);
}

function formatTime(time: number) {
  const totalMilliseconds = Math.floor(time);
  const minutes = Math.floor((totalMilliseconds % 3600_000) / 60_000);
  const seconds = Math.floor((totalMilliseconds % 60_000) / 1000);
  const milliseconds = totalMilliseconds % 1000;

  return pad(minutes) + ":" + pad(seconds) + ":" + milliseconds.toString().padStart(3, "0");
}

function getDashArray(percent) {
  const circumference = 2 * Math.PI * 90;
  const dashArray = circumference * (percent / 100);
  return `${dashArray} ${circumference}`;
}

function pad(value: number) {
  return value < 10 ? "0" + value : value;
}

function stopBrew() {
  stepIndex.value = 0;
  resetTimer()
}
</script>

<template>
  <div class="wrapper">
    <svg viewBox="0 0 200 200">
      <!-- Background Circle -->
      <circle cx="100" cy="100" r="90" fill="none" stroke="oklch(70 10 60 / 0.05)" stroke-width="8"></circle>
      <!-- Progress Arc -->
      <circle id="progressArc" cx="100" cy="100" r="90" fill="none" stroke="oklch(70 10 60)" stroke-width="8"
              stroke-linecap="round"
              :stroke-dasharray="getDashArray((elapsedTime / step.time) * 100)"></circle>
    </svg>
    <div class="time segmented-1">{{ formatTime(elapsedTime) }}</div>
    <div class="steps">
      <TransitionGroup name="list" tag="div">
        <div :key="stepIndex" class="step">
          {{ step.description }}
        </div>
        <div class="nextStep" v-if="nextStep" :key="stepIndex + 1">
          {{ nextStep.description }}
        </div>
      </TransitionGroup>
    </div>
  </div>
  <div>
    <button @click="startTimer">Start</button>
    <button @click="stopTimer">Stop</button>
    <button @click="stopBrew">reset</button>
  </div>
</template>

<style scoped>

.wrapper {
  position: relative;
  width: 500px;
  height: 500px;
}

.time {
  position: absolute;
  left: 50%;
  top: 70%;
  transform: translate(-50%, -50%);
  font-family: 'Segmented', sans-serif;
  font-variant-numeric: tabular-nums;
  font-size: 50px;
}

@font-palette-values --segmented-1 {
  font-family: Segmented;
  override-colors: 0 oklch(70 10 60), 1 oklch(70 10 60 / 0.05);
}

.segmented-1 {
  font-family: Segmented;
  font-palette: --segmented-1;
  /*font-feature-settings: 'ss${formattedCurrentStyleNumber}' on;*/
  font-variation-settings: 'wght' 600, 'wdth' 100, 'slnt' 5;
}

.steps {
  position: absolute;
  left: 50%;
  top: 40%;
  transform: translate(-50%, -50%);
}

.step {
  font-size: 30px;
  color: oklch(70 10 60);
}

.nextStep {
  color: oklch(70 10 60 / 0.05);
}

.list-move, /* apply transition to moving elements */
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
}

/* ensure leaving items are taken out of layout flow so that moving
   animations can be calculated correctly. */
.list-leave-active {
  position: absolute;
}
</style>
