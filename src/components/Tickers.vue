<template>
  <div>
    <hr class="w-full border-t border-gray-600 my-4" />
    <div>
      <button
        v-if="page > 1"
        class="my-4 mx-2 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
        @click="page = page - 1"
      >
        Назад
      </button>
      <button
        v-if="hasNextPage"
        class="my-4 mx-2 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
        @click="page = page + 1"
      >
        Вперед
      </button>
      <div>Фильтр: <input v-model="filter" /></div>
    </div>
    <hr class="w-full border-t border-gray-600 my-4" />
    <dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
      <!-- border-4 -->
      <div
        v-for="tick in filtredTickers()"
        :key="`ticker-${tick.id}`"
        @click="selectTicker(tick)"
        :class="{ 'border-4': selectedTicker === tick }"
        class="bg-white overflow-hidden shadow rounded-lg border-purple-800 border-solid cursor-pointer"
      >
        <div class="px-4 py-5 sm:p-6 text-center">
          <dt class="text-sm font-medium text-gray-500 truncate">
            {{ tick.name }} - USD
          </dt>
          <dd class="mt-1 text-3xl font-semibold text-gray-900">
            {{ tick.price }}
          </dd>
        </div>
        <div class="w-full border-t border-gray-200"></div>
        <button
          @click.stop="removeTicker(tick)"
          class="flex items-center justify-center font-medium w-full bg-gray-100 px-4 py-4 sm:px-6 text-md text-gray-500 hover:text-gray-600 hover:bg-gray-200 hover:opacity-20 transition-all focus:outline-none"
        >
          <ico-remove /> Удалить
        </button>
      </div>
    </dl>
    <hr class="w-full border-t border-gray-600 my-4" />
  </div>
</template>

<script>
import icoRemove from "../icons/reomve.vue";

export default {
  components: { icoRemove },
  props: {
    tickers: { type: Array, default: () => [] },
    selectedTicker: { type: Object, default: () => {} },
  },
  data() {
    return {
      page: 1,
      filter: "",
      hasNextPage: true,
    };
  },
  created() {
    const windowData = Object.fromEntries(
      new URL(window.location).searchParams.entries()
    );
    const { filter, page } = windowData;
    this.filter = filter || this.filter;
    this.page = page || this.page;
  },
  watch: {
    filter() {
      this.page = 1;
      this.updateUrl();
    },
    page() {
      this.updateUrl();
    },
  },
  methods: {
    filtredTickers() {
      const start = (this.page - 1) * 6;
      const end = this.page * 6;
      const filtredList = this.tickers.filter((ticker) =>
        ticker.name.includes(this.filter.toUpperCase())
      );

      this.hasNextPage = filtredList.length > end;

      return filtredList.slice(start, end);
    },
    removeTicker(tick) {
      this.$emit("remove", tick);
    },
    selectTicker(tick) {
      this.$emit("select", tick);
    },
    resetFilter() {
      this.filter = "";
    },
    updateUrl() {
      window.history.pushState(
        null,
        document.title,
        `${window.location.pathname}?filter=${this.filter}&page=${this.page}`
      );
    },
  },
};
</script>
