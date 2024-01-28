<template>
  <div class="grid grid-cols-3 w-full h-screen">
    <div class="p-8">
      <div class="text-lg font-medium uppercase">Overview</div>
      <ul>
        <li v-for="(employee, i) in employeesWorkingDays" :key="i">
          {{ employee.name }}: {{ employee.days }}
        </li>
      </ul>
    </div>

    <div class="col-span-2 grid grid-cols-7 divide-x divide-y">
      <div v-for="item in month" :key="item.day" class="flex gap-4 p-4 odd:bg-slate-100">
        <div class="font-medium">{{ item.day }}</div>
        <div class="">
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
import { computed } from 'vue'

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

const DAYS = 31
const EMPLOYEES_PER_DAY = 4

const month = computed(() => {
  const planning = []

  for (let index = 1; index <= DAYS; index++) {
    const day = {
      day: index,
      employees: [],
      isWeekend: index % 7 === 0
    }

    if (day.isWeekend) {
      day.employees = [...planning[index - 2].employees]
    } else {
      // Use spread operator to create a new array and not mutate the original one
      const shuffledEmployees = shuffle([...EMPLOYEES])

      shuffledEmployees.map((employee) => {
        // const isWorking = Math.random() > 0.5

        const averageWorkingDays = Math.round((DAYS / EMPLOYEES.length) * EMPLOYEES_PER_DAY)

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
