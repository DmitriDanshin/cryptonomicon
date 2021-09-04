<template>
  <button
    :disabled="disabled"
    :class="{
      'opacity-50': disabled,
    }"
    class="add-btn"
  >
    <v-svg name="add" />
    Добавить
  </button>
</template>
<script>
import VSvg from "@/components/v-svg";

export default {
  components: { VSvg },
  props: {
    disabled: {
      type: Boolean,
      required: false,
      default: false,
    },
  },
};
</script>
<style scoped>
.add-btn {
  @apply my-4
  inline-flex
  items-center
  py-2
  px-4
  border border-transparent
  shadow-sm
  text-sm
  leading-4
  font-medium
  rounded-full
  text-white
  bg-gray-600
  hover:bg-gray-700
  transition-colors
  duration-300
  focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500;
}
</style>
