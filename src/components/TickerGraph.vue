<template>
  <section class="relative mb-3">
    <h3 class="text-lg leading-6 font-medium text-gray-900 my-8">
      {{ ticker.name }} - USD
    </h3>
    <div
      ref="graph"
      class="flex items-end border-gray-600 border-b border-l h-64"
    >
      <div
        v-for="(bar, idx) in normalizedGraph"
        :key="idx"
        :style="{ height: `${bar}%`, width: `${widthOfBar}px` }"
        class="bg-purple-800 border"
      ></div>
    </div>

    <button @click="hideGraph" type="button" class="absolute top-0 right-0">
      <v-svg name="graph-close" />
    </button>
  </section>
</template>

<script>
import VSvg from "@/components/v-svg";

export default {
  name: "TickerGraph",
  components: { VSvg },
  props: {
    ticker: {
      type: Object,
      required: false,
    },

    graph: {
      type: Array,
      required: false,
    },
  },

  data() {
    return {
      widthOfBar: 38,
      maxGraphElements: 1,
    };
  },
  mounted() {},
  methods: {
    hideGraph() {
      this.$emit("hide-graph");
    },

    calculateGraphMaxElements() {
      if (!this.$refs.graph) {
        return;
      }

      this.maxGraphElements = this.$refs.graph.clientWidth / this.widthOfBar;
    },
  },

  computed: {
    normalizedGraph() {
      const maxValue = Math.max(...this.graph);
      const minValue = Math.min(...this.graph);

      if (maxValue === minValue) {
        return this.graph.map(() => 50);
      }

      return this.graph.map(
        (price) => 5 + ((price - minValue) * 95) / (maxValue - minValue)
      );
    },
  },
};
</script>

<style scoped></style>
