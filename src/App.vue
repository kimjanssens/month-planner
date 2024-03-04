<template>
  <div class="grid grid-cols-3 w-full h-screen">
    <div class="p-8">
      <h2 class="text-lg font-medium uppercase mb-4">Planning Info</h2>

      <calendar-form v-model="form" class="mb-20" />

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
        :active-user="activeUser"
        :employees="employees"
        @selected-day="toggleActiveUserWorkday"
      />
    </div>
  </div>
</template>

<script setup>
import { computed, reactive, ref } from 'vue'

import CalendarDay from '@/components/CalendarDay.vue'
import CalendarForm from '@/components/CalendarForm.vue'
import { DAYS_OF_THE_WEEK } from '@/utils/constants'

const employees = ref([
  { name: 'Lilian', workDays: [], holidays: [] },
  { name: 'Yasmine', workDays: [], holidays: [] },
  { name: 'Fran', workDays: [], holidays: [] },
  { name: 'Francoise', workDays: [], holidays: [] },
  { name: 'Kelly', workDays: [], holidays: [] },
  { name: 'Dina', workDays: [], holidays: [] },
  { name: 'Shari', workDays: [], holidays: [] },
  { name: 'Dana', workDays: [], holidays: [] },
  { name: 'Nora', workDays: [], holidays: [] }
])

const form = reactive({
  // Default value for input[type="month"]
  // Default format is YYYY-MM and we need to add a leading zero to the month if it's less than 10
  datepicker: `${new Date().getFullYear()}-${new Date().getMonth() < 10 ? '0' + (new Date().getMonth() + 1) : new Date().getMonth() + 1}`,
  slots: 4
})
const refreshTable = ref(0)
const activeUser = ref(null)

// Get the days of the selected month.
const days = computed(() => {
  const days = []
  const firstDateOfTheMonth = new Date(form.datepicker)

  while (firstDateOfTheMonth.getMonth() === new Date(form.datepicker).getMonth()) {
    days.push(new Date(firstDateOfTheMonth))
    firstDateOfTheMonth.setDate(firstDateOfTheMonth.getDate() + 1)
  }

  return days
})

// Create a new array of objects with the days of the month and the employees working on that day.
// This is the main function that calculates the planning
const month = computed(() => {
  const planning = []
  refreshTable.value

  // Use spread operator to create a new array and not mutate the original one
  const partTimeEmployees = employees.value.filter((employee) => employee.workDays.length > 0)
  const fullTimeEmployees = employees.value.filter((employee) => !employee.workDays.length)
  
  days.value.map((d) => {
    const day = {
      dayNumber: d.getDate(),
      employees: [],
      dayOfTheWeek: d.getDay(),
      isSunday: d.getDay() % 7 === 0,
      isWeekend: d.getDay() % 6 === 0 || d.getDay() % 7 === 0
    }

    if (day.isSunday && day.dayNumber > 1) {
      // Copy the employees from saturday to sunday.
      // Sunday must have the same employees as Saturday
      day.employees = [...planning[day.dayNumber - 2].employees]
    } else {
      // Part-time employees are added first to the planning
      // Shuffle the array of full-time employees to make the result more random
      const shuffledEmployees = [...partTimeEmployees, ...shuffle(fullTimeEmployees)]

      shuffledEmployees.map((employee) => {
        let isWorking = false

        // If an employee has a holiday, skip him
        if (employee.holidays.includes(day.dayNumber)) {
          return
        }

        // Calculate the average working days for employees
        const averageWorkingDays = Math.ceil(
          (days.value.length * form.slots - totalPartTimeWorkingDays.value) /
            fullTimeEmployees.length
        )

        // Make sure a full-time employee doesn't work more days than the average
        isWorking =
          planning.filter((d) => d.employees.includes(employee.name)).length < averageWorkingDays

        // First check if employee has workDays because this is only for part-time employees.
        // Then check if the current day is included in the workDays
        if (employee.workDays.length > 0) {
          if (employee.workDays.includes(day.dayNumber)) {
            isWorking = true
          } else {
            isWorking = false
          }
        }

        // If the day is a weekend, only add 4 employees
        if (
          day.employees.length < (day.isWeekend ? 4 : form.slots) &&
          !day.employees.includes(employee.name) &&
          isWorking
        ) {
          day.employees.push(employee.name)
        }
      })
    }

    planning.push(day)
  })

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

// First click adds the employee to the day.
// Second click removes the employee from the day.
// Third click resets  the day.
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
      employee.workDays.push(day.dayNumber)
    }

    return employee
  })
}
</script>
