<template>
  <div
    class="p-4 bg-slate-50 odd:bg-slate-200 relative"
    @click="$emit('selected-day', day)"
  >
    <div
      class="font-medium text-7xl opacity-20 absolute right-0 bottom-0"
    >
      {{ day.dayNumber }}
    </div>
  </div>
</template>

<script setup>
defineProps({
  day: {
    type: Object,
    required: true
  },
})

defineEmits(['selected-day'])
</script>
