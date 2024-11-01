<template>
  <div class="grid grid-cols-3 w-full h-screen">
    <div class="flex flex-col gap-20 p-8">
      <!-- Planning Info -->
      <div>
        <h2 class="text-lg font-medium uppercase mb-4">Planning Info</h2>
        <calendar-form v-model="form" />
      </div>

      <!-- Employees Overview -->
      <div>
        <h2 class="text-lg font-medium uppercase mb-4">Employees Overview</h2>
        <ul v-if="form.employees.length" class="flex flex-col">
          <li
            v-for="(employee, i) in form.employees"
            :key="employee"
            class="flex items-center justify-between gap-4 p-4 cursor-pointer"
            :class="{ 'bg-slate-50': i % 2 === 0, 'bg-slate-100': i % 2 !== 0 }"
            @click="toggleActiveEmployee(employee)"
          >
            {{ employee }}
            <span class="text-xs bg-black text-white px-2 py-1 rounded-full">
              {{ employee === activeEmployee ? 'Active' : 'Inactive' }}
            </span>
          </li>
        </ul>
        <p v-else>No employees have been added yet. Please add employees to start planning.</p>
      </div>

      <!-- Day Details -->
      <div v-if="activeDay">
        <h2 class="text-lg font-medium uppercase mb-4">Day Details</h2>
        <pre>{{ activeDay }}</pre>
        <DayForm v-model="activeDay" :employees="form.employees" />
      </div>

      <button
        type="button"
        @click="calculatePlanning"
        class="flex items-center justify-center w-full py-3 bg-black text-white uppercase text-sm tracking-wider font-medium"
      >
        Calculate Planning
      </button>
    </div>

    <!-- Calendar -->
    <div class="col-span-2 grid grid-cols-7">
      <div
        v-for="day in DAYS_OF_THE_WEEK"
        :key="day"
        class="text-white bg-slate-800 pl-4 flex items-center"
      >
        {{ day.name }}
      </div>

      <calendar-day
        v-for="day in planning"
        :key="day.dayNumber"
        :day="day"
        :days-of-the-week="DAYS_OF_THE_WEEK"
        :active-employee="activeEmployee"
        :active-day="activeDay"
        :employees="form.employees"
        @selected-day="setActiveDay"
      />
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, reactive, ref, watch } from 'vue'

import CalendarDay from '@/components/CalendarDay.vue'
import CalendarForm from '@/components/CalendarForm.vue'
import { DAYS_OF_THE_WEEK } from '@/utils/constants'
import DayForm from './components/DayForm.vue'

const form = reactive({
  // Default value for input[type="month"]
  // Default format is YYYY-MM and we need to add a leading zero to the month if it's less than 9
  datepicker: `${new Date().getFullYear()}-${new Date().getMonth() < 9 ? '0' : ''}${new Date().getMonth() + 1}`,
  employees: []
})

const activeEmployee = ref(null)
const activeDay = ref(null)

const planning = ref([])
const fullEmployees = ref([])

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

const setActiveDay = (day) => {
  // Toggle active day
  if (activeDay.value && activeDay.value.date === day.date) {
    activeDay.value = null
  } else {
    activeDay.value = day
  }

  // Toggle active user in employeesNotWorking
  if (!activeEmployee.value) return
  if (day.employeesNotWorking.includes(activeEmployee.value)) {
    day.employeesNotWorking = day.employeesNotWorking.filter(
      (employee) => employee !== activeEmployee.value
    )
  } else {
    day.employeesNotWorking.push(activeEmployee.value)
  }
}

const toggleActiveEmployee = (employee) => {
  if (activeEmployee.value === employee) {
    activeEmployee.value = null
  } else {
    activeEmployee.value = employee
  }
}

const calculatePlanning = () => {
  planning.value = days.value.map((d, i) => {
    const day = {
      date: d,
      dayNumber: d.getDate(),
      employeesNotWorking: [],
      dayOfTheWeek: d.getDay(),
      isSunday: d.getDay() % 7 === 0,
      isWeekend: d.getDay() % 6 === 0 || d.getDay() % 7 === 0,
      ob: null
    }

    if (!form.employees.length) return day

    day.ob = form.employees[day.dayNumber % form.employees.length]

    console.log(day)

    // Sunday ob is always te same as the previous day
    if (day.isSunday) {
      day.ob = planning.value[i - 1].ob
    }
  })

  console.log(planning.value)
}

watch(form.employees, () => {
  fullEmployees.value = form.employees
})

onMounted(() => {
  calculatePlanning()
})
</script>
