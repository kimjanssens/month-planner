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
        required
        pattern="[0-9]{4}-[0-9]{2}"
      />
    </div>

    <div class="flex items-center gap-4 mb-4">
      <label for="employees" class="flex-shrink-0">Employees:</label>
      <multiselect
        v-model="model.employees"
        :options="options"
        :multiple="true"
        :close-on-select="false"
        :taggable="true"
        @tag="handleEmployee"
        tag-placeholder="Add to employees"
        placeholder="Type to add employee"
        id="employees"
      ></multiselect>
    </div>
  </form>
</template>

<script setup>
import Multiselect from 'vue-multiselect'
import { ref } from 'vue'

const model = defineModel()
const options = ref([])

const handleEmployee = (employee) => {
  if (!options.value.find((option) => option === employee)) {
    options.value.push(employee)
    model.value.employees.push(employee)
  }
}
</script>
