<template>
  <section>
    <div class="flex">
      <add-field v-model="ticker" @keydown.enter="add" />
    </div>
    <add-button @click="add" :disabled="disabled"></add-button>
  </section>
</template>

<script>
import AddButton from "./AddButton";
import AddField from "@/components/AddField";

export default {
  name: "AddTicker",
  components: { AddField, AddButton },

  props: {
    disabled: {
      type: Boolean,
      required: false,
      default: false,
    },
  },

  data() {
    return {
      ticker: "",
    };
  },

  methods: {
    add() {
      this.$emit("add-ticker", this.ticker);
      this.ticker = "";
    },
  },
};
</script>

<style scoped></style>
