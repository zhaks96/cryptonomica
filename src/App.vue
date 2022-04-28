<template>
  <div class="container mx-auto flex flex-col items-center bg-gray-100 p-4">
    <Loader v-if="load" />
    <div class="container">
      <section>
        <div class="flex">
          <div class="max-w-xs">
            <label for="wallet" class="block text-sm font-medium text-gray-700">
              Тикер
            </label>
            <div class="mt-1 relative rounded-md shadow-md">
              <input
                v-model="ticker"
                type="text"
                name="wallet"
                id="wallet"
                class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
                placeholder="Например DOGE"
                @keydown.enter="checkTicker"
              />
            </div>
            <div
              v-if="ticker"
              class="flex bg-white shadow-md p-1 rounded-md shadow-md flex-wrap"
            >
              <span
                v-for="cryptoCoin in coinlistSlice.slice(0, 4)"
                :key="`cryptoCoin-${cryptoCoin.Id}`"
                @click="selectCryptoCoin(cryptoCoin)"
                class="inline-flex items-center px-2 m-1 rounded-md text-xs font-medium bg-gray-300 text-gray-800 cursor-pointer"
              >
                {{ cryptoCoin.Symbol }}
              </span>
            </div>
            <div v-if="isExist" class="text-sm text-red-600">
              Такой тикер уже добавлен
            </div>
          </div>
        </div>
        <button
          @click="checkTicker"
          :disabled="!ticker"
          type="button"
          class="my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
        >
          <ico-plus /> Добавить
        </button>
      </section>

      <template v-if="tickers.length">
        <Tickers
          :tickers="tickers"
          @remove="removeTicker"
          @select="selectTicker"
        />
      </template>
      <GraphTicker
        :selectedTicker="selectedTicker"
        :graphTickers="graphTickers"
        @reset="resetTicker"
      />
    </div>
  </div>
</template>

<script>
import Loader from "./components/Loader.vue";
import icoPlus from "./icons/plus.vue";
import Tickers from "./components/Tickers.vue";
import GraphTicker from "./components/GraphTicker.vue";

export default {
  name: "App",
  components: {
    Tickers,
    Loader,
    icoPlus,
    GraphTicker,
  },
  data() {
    return {
      ticker: "",
      tickers: [],
      selectedTicker: null,
      graphTickers: [],
      load: true,
      isExist: false,
      cryptoCoinlist: [],
      filtredCryptoCoinlist: [],
    };
  },
  watch: {
    ticker(v) {
      this.isExist = false;
      this.filtredCryptoCoinlist = this.cryptoCoinlist.filter(
        (x) => x.Symbol.indexOf(v.toUpperCase()) !== -1
      );
    },
  },
  computed: {
    coinlistSlice() {
      return this.ticker ? this.filtredCryptoCoinlist : this.cryptoCoinlist;
    },
  },
  mounted() {
    this.coinlist();
  },
  methods: {
    async coinlist() {
      this.load = true;
      const url =
        "https://min-api.cryptocompare.com/data/all/coinlist?summary=true";
      const coinList = await fetch(url).finally(() => {
        setTimeout(() => (this.load = false), 1500);
      });
      const data = await coinList.json();
      const { Data } = data;
      this.cryptoCoinlist = Object.keys(Data).map((key) => Data[key]);
    },
    checkTicker() {
      const existTicker = this.tickers.filter(
        (t) => t.name === this.ticker.toUpperCase()
      );
      if (existTicker.length > 0) {
        this.isExist = true;
      } else {
        this.addTicker();
        this.isExist = false;
      }
    },
    selectCryptoCoin(cryptoCoin) {
      this.ticker = cryptoCoin.Symbol;
      setTimeout(() => {
        this.checkTicker();
      }, 0);
    },
    addTicker() {
      if (this.ticker) {
        const currentTicker = {
          name: this.ticker.toUpperCase(),
          price: "-",
        };
        this.tickers.push(currentTicker);
        setInterval(async () => {
          const f = await fetch(
            `https://min-api.cryptocompare.com/data/price?fsym=${currentTicker.name}&tsyms=USD&api_key=${process.env.API_KEY}`
          );
          const data = await f.json();
          const { USD } = data;
          this.tickers.find(
            (ticker) => ticker.name === currentTicker.name
          ).price = USD > 1 ? USD.toFixed(2) : USD.toPrecision(2);
          if (this.selectedTicker?.name === currentTicker.name) {
            this.graphTickers.push(USD);
          }
        }, 5000);
        this.ticker = "";
      }
    },
    selectTicker(ticker) {
      this.selectedTicker = ticker;
      this.graphTickers = [];
    },
    removeTicker(tick) {
      this.tickers = this.tickers.filter((t) => t !== tick);
    },
    resetTicker() {
      this.selectedTicker = null;
    },
  },
};
</script>
