<template>
  <form>
    <div class="flex items-center gap-4 mb-4">
      <label for="ob" class="flex-shrink-0">OB:</label>
      <multiselect
        v-model="model.ob"
        :options="employees"
        placeholder="Select employee"
        id="ob"
      ></multiselect>
    </div>
  </form>
</template>

<script setup>
import Multiselect from 'vue-multiselect'

defineProps({
  employees: {
    type: Array,
    required: true
  }
})

const model = defineModel()
</script>
