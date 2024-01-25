<template>
  <pre>{{ JSON.stringify(planning, null, 2) }}</pre>
</template>

<script setup>
import { computed } from 'vue'

const EMPLOYEES = [
  'Lilian',
  'Yasmine',
  'Fran',
  'Francoise',
  'Kelly',
  'Dina',
  'Shari',
  'Dana',
  'Nora'
]

const DAYS = 31

const planning = computed(() => {
  const month = []

  for (let index = 1; index <= DAYS; index++) {
    const day = {
      day: index,
      employees: []
    }

    EMPLOYEES.map((employee) => {
      const isWorking = Math.random() > 0.5

      if (isWorking) {
        day.employees.push(employee)
      }
    })

    month.push(day)
  }

  return month
})
</script>

<style scoped lang="scss"></style>
