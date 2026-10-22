<template>
  <div class="countdown" role="timer" :aria-label="label">
    <div class="countdown__inner" aria-hidden="true">
      <template v-if="showDays">
        <div class="countdown__group">
          <FlipDigit :value="days" />
          <span class="countdown__label">วัน</span>
        </div>
        <span class="countdown__sep">:</span>
      </template>

      <div class="countdown__group">
        <FlipDigit :value="hours" />
        <span class="countdown__label">ชั่วโมง</span>
      </div>
      <span class="countdown__sep">:</span>

      <div class="countdown__group">
        <FlipDigit :value="minutes" />
        <span class="countdown__label">นาที</span>
      </div>
      <span class="countdown__sep">:</span>

      <div class="countdown__group">
        <FlipDigit :value="seconds" />
        <span class="countdown__label">วินาที</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import FlipDigit from './FlipDigit.vue'

defineProps({
  days: {
    type: String,
    default: '00',
  },
  hours: {
    type: String,
    default: '00',
  },
  minutes: {
    type: String,
    default: '00',
  },
  seconds: {
    type: String,
    default: '00',
  },
  showDays: {
    type: Boolean,
    default: false,
  },
  label: {
    type: String,
    default: '',
  },
})
</script>

<style scoped>
.countdown {
  display: flex;
  justify-content: center;
}

.countdown__inner {
  display: flex;
  align-items: flex-start;
  gap: clamp(4px, 1.2vw, 10px);
}

.countdown__group {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.countdown__label {
  color: var(--muted, #94a2bc);
  font-size: 0.7rem;
  font-weight: 500;
  line-height: 1.4;
  margin-top: -.5em
}

.countdown__sep {
  padding-top: 0.55em;
  color: rgba(255, 255, 255, 0.32);
  font-size: 1.5rem;
  font-weight: 700;
  line-height: 1;
}
</style>
