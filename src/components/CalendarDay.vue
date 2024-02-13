<template>
  <div
    class="p-4 bg-slate-50 odd:bg-slate-200 relative"
    :class="{
      [daysOfTheWeek[firstDayOfTheMonth].tailwind]: day.dayNumber === 1,
      'cursor-pointer': activeUser
    }"
    @click="$emit('selected-day', day)"
  >
    <div
      class="font-medium text-7xl opacity-20 absolute right-0 bottom-0"
      :class="{
        'text-green-400 opacity-40': day.employees.includes(activeUser?.name),
        'text-red-400 opacity-40': employees.find(
          (e) =>
            e.name === activeUser?.name &&
            e.holidays.includes(day.dayNumber) &&
            !e.workingDays.includes(day.dayNumber)
        )
      }"
    >
      {{ day.dayNumber }}
    </div>

    <div>
      <ul>
        <li v-for="employee in day.employees" :key="employee">
          {{ employee }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
defineProps({
  day: {
    type: Object,
    required: true
  },
  daysOfTheWeek: {
    type: Object,
    required: true
  },
  firstDayOfTheMonth: {
    type: Number,
    required: true
  },
  activeUser: {
    type: Object,
    default: null
  },
  employees: {
    type: Array,
    required: true
  }
})

defineEmits(['selected-day'])
</script>

<style lang="scss" scoped></style>
