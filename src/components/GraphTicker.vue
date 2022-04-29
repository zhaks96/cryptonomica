<template>
  <section v-if="selectedTicker" class="relative">
    <h3 class="text-lg leading-6 font-medium text-gray-900 my-8">
      {{ selectedTicker.name }}
    </h3>
    <div class="flex items-end border-gray-600 border-b border-l h-64">
      <div
        v-for="(bar, index) in normalizedGraph"
        :key="`bar-${index}`"
        :style="{ height: `${bar}%` }"
        class="bg-purple-800 border w-10"
      ></div>
    </div>
    <button
      type="button"
      class="absolute top-0 right-0"
      @click="$emit('reset')"
    >
      <ico-close />
    </button>
  </section>
</template>

<script>
import icoClose from "../icons/close.vue";
export default {
  components: {
    icoClose,
  },
  props: {
    selectedTicker: { type: Object, default: () => null },
    graphTickers: { type: Object, default: () => null },
  },
  computed: {
    normalizedGraph() {
      const maxValue = Math.max(...this.graphTickers);
      const minValue = Math.min(...this.graphTickers);

      if (maxValue === minValue) {
        return this.graph.map(() => 50);
      }

      return this.graphTickers.map(
        (price) => 5 + ((price - minValue) * 95) / (maxValue - minValue)
      );
    },
  },
};
</script>
