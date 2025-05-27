<template>
  <div ref="tick" class="tick">
    <div data-repeat="true" aria-hidden="true">
      <span data-view="flip"></span>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, onUnmounted } from 'vue';
import Tick from "@pqina/flip";
import "@pqina/flip/dist/flip.min.css";

const props = defineProps({
  value: {
    type: [String, Number], // Assuming value can be string or number based on typical usage
    default: 0
  }
});

const tick = ref(null); // Reference to the DOM element
let _tickInstance = null; // To hold the Tick instance

onMounted(() => {
  _tickInstance = Tick.DOM.create(tick.value, {
    value: props.value
  });
});

watch(() => props.value, (newValue) => {
  if (_tickInstance) {
    _tickInstance.value = newValue;
  }
});

onUnmounted(() => {
  if (_tickInstance) {
    Tick.DOM.destroy(tick.value);
  }
});
</script>

<style scoped>
.tick {
  font-size: 2.5em;
}

/* Media query for screens smaller than 480px (common mobile breakpoint) */
@media (max-width: 480px) {
.tick {
  font-size: 2rem;
}
}
</style>