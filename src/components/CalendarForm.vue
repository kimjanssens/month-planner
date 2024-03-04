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
const model = defineModel()
</script>

<style lang="scss" scoped></style>
