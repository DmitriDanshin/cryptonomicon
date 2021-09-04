<template>
  <svg :width="width" :height="height">
    <use :href="src"></use>
  </svg>
</template>

<script>
export default {
  name: "v-svg",
  props: {
    name: {
      type: String,
      required: true,
    },
    width: {
      type: String,
      default: "30px",
    },
    height: {
      type: String,
      default: "30px",
    },
  },
  computed: {
    src() {
      return require("../assets/sprite.svg") + `#${this.name}`;
    },
  },
};
</script>

<style scoped></style>
