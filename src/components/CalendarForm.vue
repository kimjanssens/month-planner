<template>
  <form>
    <div class="flex items-center gap-4 mb-4">
      <label for="datepicker" class="flex-shrink-0">Datum:</label>
      <input
        type="month"
        id="datepicker"
        name="datepicker"
        lang="nl"
        v-model="model.datepicker"
        class="w-full"
      />
    </div>

    <div class="flex items-center gap-4 mb-4">
      <label for="slots" class="flex-shrink-0"># of shifts:</label>
      <input type="number" id="slots" name="slots" v-model="model.slots" class="w-full" />
    </div>

    <div class="flex items-center gap-4 mb-4">
      <label for="employees" class="flex-shrink-0">Employees:</label>
      <multiselect
        v-model="model.employees"
        :options="options"
        :multiple="true"
        :close-on-select="false"
        :taggable="true"
        @tag="addEmployee"
        tag-placeholder="Add to employees"
        placeholder="Type to add employee"
        id="employees"
        label="name"
      ></multiselect>
    </div>

    <button
      type="button"
      @click="refreshTable++"
      class="flex items-center justify-center w-full py-3 bg-black text-white uppercase text-sm tracking-wider font-medium"
    >
      Calculate
    </button>
  </form>
</template>

<script setup>
import Multiselect from 'vue-multiselect'
import { ref } from 'vue'

const model = defineModel()
const options = ref([])

const addEmployee = (newEmployee) => {
  model.value.employees.push({
    name: newEmployee,
    workDays: [],
    holidays: []
  })
  options.value.push({
    name: newEmployee,
    workDays: [],
    holidays: []
  })
}
</script>
