<template>
  <div
    class="p-4 bg-slate-50 odd:bg-slate-200 relative cursor-pointer"
    :class="{
      [DAYS_OF_THE_WEEK[day.dayOfTheWeek]?.tailwind]: day.dayNumber === 1,
      'bg-gradient-to-br from-green-400 from-10%% to-transparent': day.date === activeDay?.date
    }"
    @click="$emit('selected-day', day)"
  >
    <div class="flex flex-col gap-2">
      <span>OB: {{ day.ob }}</span>
      <span>LA:</span>
      <span>KO:</span>
      <span>BT:</span>
      <span>EX:</span>
    </div>

    <div
      class="font-medium text-7xl opacity-20 absolute right-0 bottom-0"
      :class="{
        'text-red-400 opacity-40': day.employeesNotWorking.includes(activeEmployee)
      }"
    >
      {{ day.dayNumber }}
    </div>
  </div>
</template>

<script setup>
import { DAYS_OF_THE_WEEK } from '@/utils/constants'

defineProps({
  day: {
    type: Object,
    required: true
  },
  activeEmployee: {
    type: String,
    required: false
  },
  activeDay: {
    type: Object,
    required: false
  }
})

defineEmits(['selected-day'])
</script>

<style lang="scss" scoped></style>
