<template>
  <div class="grid grid-cols-3 w-full h-screen">
    <div class="p-8">
      <h2 class="text-lg font-medium uppercase mb-4">Planning Info</h2>
      <form class="mb-10">
        <div class="mb-4">
          <label for="days" class="mr-4">Days:</label>
          <input type="number" id="days" name="days" v-model="days" />
        </div>
        <div>
          <label for="firstDayOfTheMonth" class="mr-4">First day of the month:</label>
          <select v-model="firstDayOfTheMonth" id="firstDayOfTheMonth">
            <option :value="0">Monday</option>
            <option :value="1">Tuesday</option>
            <option :value="2">Wednesday</option>
            <option :value="3">Thursday</option>
            <option :value="4">Friday</option>
            <option :value="5">Saturday</option>
            <option :value="6">Sunday</option>
          </select>
        </div>
      </form>

      <h2 class="text-lg font-medium uppercase mb-4">Overview</h2>
      <ul>
        <li v-for="(employee, i) in employeesWorkingDays" :key="i">
          {{ employee.name }}: {{ employee.days }}
        </li>
      </ul>
    </div>

    <div class="col-span-2 grid grid-cols-7">
      <div
        v-for="day in DAYS_OF_THE_WEEK"
        :key="day"
        class="text-white bg-slate-800 pl-4 flex items-center"
      >
        {{ day.name }}
      </div>

      <div
        v-for="item in month"
        :key="item.day"
        class="p-4 bg-slate-50 odd:bg-slate-200"
        :class="{ [DAYS_OF_THE_WEEK[firstDayOfTheMonth].tailwind]: item.day === 1 }"
      >
        <div class="font-medium">{{ item.day }}</div>
        <div>
          <ul>
            <li v-for="employee in item.employees" :key="employee">
              {{ employee }}
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref } from 'vue'

const EMPLOYEE_STATUS = {
  FULL_TIME: 'full-time',
  PART_TIME: 'part-time'
}

const EMPLOYEES = [
  { name: 'Lilian', status: EMPLOYEE_STATUS.PART_TIME },
  { name: 'Yasmine', status: EMPLOYEE_STATUS.FULL_TIME },
  { name: 'Fran', status: EMPLOYEE_STATUS.FULL_TIME },
  { name: 'Francoise', status: EMPLOYEE_STATUS.FULL_TIME },
  { name: 'Kelly', status: EMPLOYEE_STATUS.FULL_TIME },
  { name: 'Dina', status: EMPLOYEE_STATUS.PART_TIME },
  { name: 'Shari', status: EMPLOYEE_STATUS.FULL_TIME },
  { name: 'Dana', status: EMPLOYEE_STATUS.FULL_TIME },
  { name: 'Nora', status: EMPLOYEE_STATUS.FULL_TIME }
]

const DAYS_OF_THE_WEEK = {
  0: { name: 'Monday', tailwind: 'col-start-1' },
  1: { name: 'Tuesday', tailwind: 'col-start-2' },
  2: { name: 'Wednesday', tailwind: 'col-start-3' },
  3: { name: 'Thursday', tailwind: 'col-start-4' },
  4: { name: 'Friday', tailwind: 'col-start-5' },
  5: { name: 'Saturday', tailwind: 'col-start-6' },
  6: { name: 'Sunday', tailwind: 'col-start-7' }
}

const EMPLOYEES_PER_DAY = 4

const days = ref(31)
const firstDayOfTheMonth = ref(0)

const month = computed(() => {
  const planning = []

  for (let index = 1; index <= days.value; index++) {
    const day = {
      day: index,
      employees: [],
      isWeekend: (index + Number(firstDayOfTheMonth.value)) % 7 === 0
    }

    if (day.isWeekend) {
      day.employees = [...planning[index - 2].employees]
    } else {
      // Use spread operator to create a new array and not mutate the original one
      const shuffledEmployees = shuffle([...EMPLOYEES])

      shuffledEmployees.map((employee) => {
        // const isWorking = Math.random() > 0.5

        const averageWorkingDays = Math.round((days.value / EMPLOYEES.length) * EMPLOYEES_PER_DAY)

        const isWorking =
          planning.filter((d) => d.employees.includes(employee.name)).length < averageWorkingDays

        if (
          day.employees.length < EMPLOYEES_PER_DAY &&
          !day.employees.includes(employee.name) &&
          isWorking
        ) {
          day.employees.push(employee.name)
        }
      })
    }

    planning.push(day)
  }

  return planning
})

const employeesWorkingDays = computed(() => {
  return EMPLOYEES.map((employee) => {
    return {
      name: employee.name,
      days: countWorkingDays(employee)
    }
  })
})

const countWorkingDays = (employee) => {
  let count = 0

  month.value.map((day) => {
    if (day.employees.includes(employee.name)) {
      count++
    }
  })

  return count
}

const shuffle = (array) => {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[array[i], array[j]] = [array[j], array[i]]
  }
  return array
}
</script>

<style scoped lang="scss"></style>
