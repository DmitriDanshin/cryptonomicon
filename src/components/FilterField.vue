<template>
  <div class="inline">
    Filter:
    <input @input="filtered" type="text" />
  </div>
</template>

<script>
export default {
  name: "FilterField",
  methods: {
    filtered(event) {
      this.$emit("update:modelValue", event.target.value);
    },
  },
};
</script>

<style scoped></style>
