<template>
  <div class="max-w-xs">
    <label for="wallet" class="block text-sm font-medium text-gray-700">
      Тикер {{ modelValue }}
    </label>
    <div class="mt-1 relative rounded-md shadow-md">
      <input
        @input="addTicker"
        :value="modelValue"
        type="text"
        name="wallet"
        id="wallet"
        class="
          block
          w-full
          pr-10
          border-gray-300
          text-gray-900
          focus:outline-none focus:ring-gray-500 focus:border-gray-500
          sm:text-sm
          rounded-md
        "
        placeholder="Например DOGE"
      />
    </div>
  </div>
</template>

<script>
export default {
  name: "AddField",
  props: {
    modelValue: String,
  },
  data() {
    return {
      ticker: this.modelValue,
    };
  },

  methods: {
    addTicker(event) {
      this.$emit("update:modelValue", event.target.value);
    },
  },
};
</script>

<style scoped></style>
