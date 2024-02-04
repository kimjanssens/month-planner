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
        <li
          v-for="(employee, i) in employeesWorkingDays"
          :key="i"
          class="grid grid-cols-5 p-2 bg-slate-50 odd:bg-slate-200 cursor-pointer hover:bg-blue-200"
          :class="{
            'border-l-4 border-l-blue-600 text-blue-600': employee.name === activeUser?.name
          }"
          @click="setActiveUser(employee)"
        >
          <div>{{ employee.name }}:</div>
          <div>
            {{ employee.days.length }}
          </div>
          <div class="col-span-3 flex justify-end">
            <svg
              v-if="employee.workDays.length > 0 || employee.holidays.length > 0"
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              strokeWidth="{1.5}"
              stroke="currentColor"
              class="w-6 h-6"
            >
              <path
                strokeLinecap="round"
                strokeLinejoin="round"
                d="m16.862 4.487 1.687-1.688a1.875 1.875 0 1 1 2.652 2.652L10.582 16.07a4.5 4.5 0 0 1-1.897 1.13L6 18l.8-2.685a4.5 4.5 0 0 1 1.13-1.897l8.932-8.931Zm0 0L19.5 7.125M18 14v4.75A2.25 2.25 0 0 1 15.75 21H5.25A2.25 2.25 0 0 1 3 18.75V8.25A2.25 2.25 0 0 1 5.25 6H10"
              />
            </svg>
          </div>
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

      <calendar-day
        v-for="day in month"
        :key="day.dayNumber"
        :day="day"
        :days-of-the-week="DAYS_OF_THE_WEEK"
        :first-day-of-the-month="firstDayOfTheMonth"
        :active-user="activeUser"
        :employees="employees"
        @selected-day="toggleActiveUserWorkday"
      />
    </div>
  </div>
</template>

<script setup>
import { computed, ref } from 'vue'

import CalendarDay from '@/components/CalendarDay.vue'
import { DAYS_OF_THE_WEEK } from '@/utils/constants'

const employees = ref([
  {
    name: 'Lilian',
    workDays: [2, 3, 4, 9, 10, 11, 16, 17, 18, 23, 24, 25, 30, 31],
    holidays: []
  },
  { name: 'Yasmine', workDays: [], holidays: [] },
  { name: 'Fran', workDays: [], holidays: [] },
  { name: 'Francoise', workDays: [], holidays: [] },
  { name: 'Kelly', workDays: [], holidays: [] },
  { name: 'Dina', workDays: [], holidays: [] },
  { name: 'Shari', workDays: [], holidays: [] },
  { name: 'Dana', workDays: [], holidays: [] },
  { name: 'Nora', workDays: [], holidays: [] }
])

const days = ref(31)
const firstDayOfTheMonth = ref(0)
const slots = ref(4)
const refreshTable = ref(0)
const activeUser = ref(null)

const month = computed(() => {
  const planning = []
  refreshTable.value

  // Use spread operator to create a new array and not mutate the original one
  const partTimeEmployees = employees.value.filter((employee) => employee.workDays.length > 0)
  const fullTimeEmployees = employees.value.filter((employee) => !employee.workDays.length)

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

        // If an employee has a holiday, skip him
        if (employee.holidays.includes(index)) {
          return
        }

        // Calculate the average working days for employees
        const averageWorkingDays = Math.ceil(
          (days.value * slots.value - totalPartTimeWorkingDays.value) / fullTimeEmployees.length
        )

        // Makee sure a full-time employee doesn't work more days than the average
        isWorking =
          planning.filter((d) => d.employees.includes(employee.name)).length < averageWorkingDays

        // First check if employee has workDays because this is only for part-time employees
        // Then check if the current day is included in the workDays
        if (employee.workDays.length > 0) {
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

  employees.value.map((employee) => {
    count += employee.workDays ? employee.workDays.length : 0
  })

  return count
})

const employeesWorkingDays = computed(() => {
  return employees.value.map((employee) => {
    return {
      ...employee,
      days: month.value
        .filter((day) => day.employees.includes(employee.name))
        .map((d) => d.dayNumber)
    }
  })
})

const shuffle = (array) => {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[array[i], array[j]] = [array[j], array[i]]
  }
  return array
}

const setActiveUser = (employee) => {
  if (activeUser.value?.name === employee.name) {
    activeUser.value = null
    return
  }

  activeUser.value = employee
}

const toggleActiveUserWorkday = (day) => {
  if (!activeUser.value) return

  employees.value = employees.value.map((employee) => {
    if (employee.name !== activeUser.value.name) {
      return employee
    }

    if (employee.workDays.includes(day.dayNumber)) {
      employee.workDays = employee.workDays.filter((d) => d !== day.dayNumber)
      employee.holidays.push(day.dayNumber)
    } else if (employee.holidays.includes(day.dayNumber)) {
      employee.holidays = employee.holidays.filter((d) => d !== day.dayNumber)
    } else {
      employee.workDays = [
        ...employeesWorkingDays.value.find((e) => e.name === activeUser.value.name).days,
        day.dayNumber
      ]
    }

    return employee
  })
}
</script>

<style scoped lang="scss"></style>
