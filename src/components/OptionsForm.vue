<!-- eslint-disable vue/no-mutating-props -->
<template>
  <form>
    <fieldset class="fieldset">
      <legend>Options</legend>

      <label for="width">width:</label>
      <input type="number" id="width" v-model="props.options.width">
      <br>
      <label for="height">height:</label>
      <input type="number" id="height" v-model="props.options.height">
      <br>
    </fieldset>
  </form>

</template>

<script setup>
import {defineProps} from 'vue';
const props = defineProps(['options']);

</script>

<style scoped>
.fieldset label {
  display: inline-block;
  width: 80px;
}
</style>