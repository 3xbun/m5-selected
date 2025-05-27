<template>
  <div class="countdown-container">
    <div class="flip-clock-wrapper">
      <div class="flip-segment">
        <FlipDigit :value="countdown.hours" />
        <span class="label">Hours</span>
      </div>
      <span class="separator">:</span>
      <div class="flip-segment">
        <FlipDigit :value="countdown.minutes" />
        <span class="label">Minutes</span>
      </div>
      <span class="separator">:</span>
      <div class="flip-segment">
        <FlipDigit :value="countdown.seconds" />
        <span class="label">Seconds</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, inject } from 'vue';
import dayjs from 'dayjs';
import duration from 'dayjs/plugin/duration';
import advancedFormat from 'dayjs/plugin/advancedFormat'; // For 'Z' or 'z' in format

import FlipDigit from './FlipDigit.vue'; // Import the FlipDigit component

// Extend Day.js with the plugins
dayjs.extend(duration);
dayjs.extend(advancedFormat);

// Reactive state
const targetTime = ref(null);
const currentTime = ref(dayjs());
let intervalId = null;

// Computed properties
const isButtonDisabled = computed(() => {
  return targetTime.value && currentTime.value.isBefore(targetTime.value);
});
const countdown = computed(() => {
  if (!targetTime.value || !currentTime.value) {
    return { hours: '00', minutes: '00', seconds: '00' };
  }

  const diffMs = targetTime.value.diff(currentTime.value);

  if (diffMs <= 0) {
    // If target time has passed, stop the interval and return all zeros
    if (intervalId) {
      clearInterval(intervalId);
      intervalId = null;
    }
    return { hours: '00', minutes: '00', seconds: '00' };
  }

  const durationObj = dayjs.duration(diffMs);

  // Calculate total hours
  const totalHours = Math.floor(durationObj.asHours());

  // Format to two digits for consistent display
  return {
    hours: String(totalHours).padStart(2, '0'),
    minutes: String(durationObj.minutes()).padStart(2, '0'),
    seconds: String(durationObj.seconds()).padStart(2, '0'),
  };
});

// Functions
const setTargetTime = () => {
  // Get current time in the target timezone, add 1 day, then set to noon
  targetTime.value = inject("targetTime")
};

const startTimer = () => {
  setTargetTime();

  if (isButtonDisabled.value) {
    intervalId = setInterval(() => {
      currentTime.value = dayjs();
      // The `countdown` computed property will handle clearing the interval
      // if targetTime is reached.
    }, 1000); // Update every second
  }
};

// Lifecycle hooks
onMounted(() => {
  startTimer();
});

onUnmounted(() => {
  if (intervalId) {
    clearInterval(intervalId);
  }
});
</script>

<style scoped>
.countdown-container {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 99;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 15px;
  border-radius: 8px;
}

.flip-clock-wrapper {
  width: 100vw;
  display: flex;
  justify-content: center;
  gap: 10px;
  align-items: center;
  background-color: #222;
  padding: 15px 20px;
  box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.5);
}

.flip-segment {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.label {
  font-size: 0.8em;
  color: #bbb;
  margin-top: 5px;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.separator {
  font-size: 3em;
  color: #fff;
  line-height: 1.2em;
  /* Align with digit height */
  transform: translateY(-0.1em);
  /* Adjust vertical alignment */
}

.countdown-finished {
  font-size: 1.5em;
  font-weight: bold;
  color: #28a745;
  margin-top: 1em;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
  opacity: 0.7;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  font-size: 1em;
  color: #555;
}

button:not(:disabled) {
  background-color: #007bff;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  font-size: 1em;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

button:not(:disabled):hover {
  background-color: #0056b3;
}
</style>