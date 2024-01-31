<template>
  <div class="grid grid-cols-3 w-full h-screen">
    <div class="p-8">
      <h2 class="text-lg font-medium uppercase mb-4">Planning Info</h2>
      <form class="mb-20">
        <div class="flex items-center gap-4 mb-4">
          <label for="days" class="flex-shrink-0">Days:</label>
          <input type="number" id="days" name="days" v-model="days" class="w-full" />
        </div>
        <div class="flex items-center gap-4 mb-4">
          <label for="firstDayOfTheMonth" class="flex-shrink-0">First day of the month:</label>
          <select v-model="firstDayOfTheMonth" id="firstDayOfTheMonth" class="w-full">
            <option :value="0">Monday</option>
            <option :value="1">Tuesday</option>
            <option :value="2">Wednesday</option>
            <option :value="3">Thursday</option>
            <option :value="4">Friday</option>
            <option :value="5">Saturday</option>
            <option :value="6">Sunday</option>
          </select>
        </div>
        <div class="flex items-center gap-4 mb-4">
          <label for="slots" class="flex-shrink-0"># of shifts:</label>
          <input type="number" id="slots" name="slots" v-model="slots" class="w-full" />
        </div>

        <button
          type="button"
          @click="refreshTable++"
          class="flex items-center justify-center w-full py-3 bg-black text-white uppercase text-sm tracking-wider font-medium"
        >
          Calculate
        </button>
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
        v-for="day in month"
        :key="day.dayNumber"
        class="p-4 bg-slate-50 odd:bg-slate-200 relative"
        :class="{ [DAYS_OF_THE_WEEK[firstDayOfTheMonth].tailwind]: day.dayNumber === 1 }"
      >
        <div class="font-medium text-7xl opacity-20 absolute right-0 bottom-0">
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
  {
    name: 'Lilian',
    status: EMPLOYEE_STATUS.PART_TIME,
    workDays: [2, 3, 4, 9, 10, 11, 16, 17, 18, 23, 24, 25, 30, 31]
  },
  { name: 'Yasmine', status: EMPLOYEE_STATUS.FULL_TIME },
  { name: 'Fran', status: EMPLOYEE_STATUS.FULL_TIME },
  { name: 'Francoise', status: EMPLOYEE_STATUS.FULL_TIME },
  { name: 'Kelly', status: EMPLOYEE_STATUS.FULL_TIME },
  { name: 'Dina', status: EMPLOYEE_STATUS.PART_TIME, workDays: [1, 2, 8] },
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

const days = ref(31)
const firstDayOfTheMonth = ref(0)
const slots = ref(4)
const refreshTable = ref(0)

const month = computed(() => {
  const planning = []
  refreshTable.value

  // Use spread operator to create a new array and not mutate the original one
  const partTimeEmployees = [...EMPLOYEES].filter((employee) => employee.workDays)
  const fullTimeEmployees = [...EMPLOYEES].filter((employee) => !employee.workDays)

  for (let index = 1; index <= days.value; index++) {
    const day = {
      dayNumber: index,
      employees: [],
      isWeekend: (index + Number(firstDayOfTheMonth.value)) % 7 === 0
    }

    if (day.isWeekend && index > 1) {
      day.employees = [...planning[index - 2].employees]
    } else {
      //
      // Shuffle the array of full-time employees to make the result more random
      const shuffledEmployees = [...partTimeEmployees, ...shuffle(fullTimeEmployees)]

      shuffledEmployees.map((employee) => {
        let isWorking = false

        // Calculate the average working days for full-time employees
        const averageWorkingDays = Math.ceil(
          (days.value * slots.value - totalPartTimeWorkingDays.value) / fullTimeEmployees.length
        )

        // Makee sure a full-time employee doesn't work more days than the average
        isWorking =
          planning.filter((d) => d.employees.includes(employee.name)).length < averageWorkingDays

        // First check if employee has workDays because this is only for part-time employees
        // Then check if the current day is included in the workDays
        if (employee?.workDays?.length > 0) {
          if (employee.workDays.includes(index)) {
            isWorking = true
          } else {
            isWorking = false
          }
        }

        if (
          day.employees.length < slots.value &&
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

const totalPartTimeWorkingDays = computed(() => {
  let count = 0

  EMPLOYEES.map((employee) => {
    count += employee.workDays ? employee.workDays?.length : 0
  })

  return count
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
