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
  background: #101a2e;
  box-shadow:
    0 16px 30px -18px rgba(2, 6, 23, 0.95),
    0 0 0 1px rgba(255, 255, 255, 0.1),
    inset 0 1px 0 rgba(255, 255, 255, 0.12);
}

.tick :deep(.tick-flip-panel) {
  background-color: #141f36;
  background-image: linear-gradient(180deg, #1d2b48 0%, #111b2e 100%);
  color: #f4f7ff;
}

.tick :deep(.tick-flip-panel-back:after) {
  background-image: linear-gradient(180deg, rgba(0, 0, 0, 0.35) 1px, rgba(0, 0, 0, 0.18) 0, transparent 30%);
}

.tick :deep(.tick-credits) {
  display: none;
}
</style>
