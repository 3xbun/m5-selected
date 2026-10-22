<template>
  <div ref="tick" class="tick">
    <div data-repeat="true" aria-hidden="true">
      <span data-view="flip"></span>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, onUnmounted } from 'vue'
import Tick from '@pqina/flip'
import '@pqina/flip/dist/flip.min.css'

const props = defineProps({
  value: {
    type: [String, Number],
    default: 0,
  },
})

const tick = ref(null)
let tickInstance = null

onMounted(() => {
  tickInstance = Tick.DOM.create(tick.value, {
    value: props.value,
    credits: null,
  })
})

watch(
  () => props.value,
  (newValue) => {
    if (tickInstance) {
      tickInstance.value = newValue
    }
  },
)

onUnmounted(() => {
  if (tickInstance) {
    Tick.DOM.destroy(tick.value)
    tickInstance = null
  }
})
</script>

<style scoped>
.tick {
  font-size: clamp(1.65rem, 5.2vw, 2.35rem);
  font-weight: 700;
  line-height: 1.35;
  font-variant-numeric: tabular-nums;
}

.tick :deep(.tick-flip) {
  overflow: hidden;
  border-radius: 12px;
  box-shadow:
    0 16px 30px -18px rgba(2, 6, 23, 0.95),
    inset 0 1px 0 rgba(255, 255, 255, 0.7);
}

.tick :deep(.tick-flip-panel) {
  background-color: #eef2fb;
  background-image: linear-gradient(180deg, #ffffff 0%, #e6edfa 100%);
  color: #0b1220;
}

.tick :deep(.tick-credits) {
  display: none;
}
</style>
