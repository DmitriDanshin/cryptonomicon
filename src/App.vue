<template>
  <div class="container">
    <add-ticker @add-ticker="add" :disabled="tooManyTickersAdded" />
    <div>
      <button v-if="page > 1" class="nav-button" @click="page = page - 1">
        Предыдущая
      </button>
      <button v-if="hasNextPage" @click="page = page + 1" class="nav-button">
        Следующая
      </button>
      <filter-field v-model="filter" />
    </div>
    {{ page }}
    <template v-if="tickers.length">
      <hr class="w-full border-t border-gray-600 my-4" />

      <dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
        <div
          v-for="(ticker, idx) in paginatedTickers"
          :key="idx"
          @click="select(ticker)"
          :class="{ 'border-2': selectedTicker === ticker }"
          class="card"
        >
          <div
            class="px-4 py-5 sm:p-6 text-center"
            v-bind:class="{ 'bg-red-200': ticker.price === '-' }"
          >
            <dt class="text-sm font-medium text-gray-500 truncate">
              {{ ticker.name }} - USD
            </dt>
            <dd class="mt-1 text-3xl font-semibold text-gray-900">
              {{ formatPrice(ticker.price) }}
            </dd>
          </div>
          <div class="w-full border-t border-gray-200"></div>

          <button @click.stop="handleDelete(ticker)" class="delete-button">
            <v-svg name="delete" />
            Удалить
          </button>
        </div>
      </dl>
      <hr class="w-full border-t border-gray-600 my-4 bg-gradient-to-b" />
    </template>
    <ticker-graph
      v-if="selectedTicker"
      :ticker="selectedTicker"
      :graph="graph"
      @hide-graph="selectedTicker = null"
    />
  </div>
</template>

<script>
import { subscribeToTicker, unsubscribeFromTicker } from "./API";
import { loadCryptocurrencies } from "@/API";

import AddTicker from "@/components/AddTicker";
import TickerGraph from "@/components/TickerGraph";
import VSvg from "@/components/v-svg";
import FilterField from "@/components/FilterField";

export default {
  name: "App",
  components: { FilterField, VSvg, TickerGraph, AddTicker },
  data() {
    return {
      tickers: [],
      selectedTicker: null,
      graph: [],
      examples: [],
      page: 1,
      filter: "",
      fullNameOfExamples: [],
    };
  },

  created() {
    this.getCryptocurrencies();

    const windowData = Object.fromEntries(
      new URL(window.location).searchParams.entries()
    );

    if (windowData.filter) {
      this.filter = windowData.filter;
    }

    if (windowData.page) {
      this.page = windowData.page;
    }

    const tickersData = localStorage.getItem("cryptonomicon-list");

    if (tickersData) {
      this.tickers = JSON.parse(tickersData);
      this.tickers.forEach((ticker) => {
        subscribeToTicker(ticker.name, (newPrice) =>
          this.updateTicker(ticker.name, newPrice)
        );
      });
    }
  },

  computed: {
    tooManyTickersAdded() {
      return this.tickers.length > 10;
    },

    startIndex() {
      return (this.page - 1) * 6;
    },

    endIndex() {
      return this.page * 6;
    },

    filteredTickers() {
      return this.tickers.filter((ticker) => ticker.name.includes(this.filter));
    },

    paginatedTickers() {
      return this.filteredTickers.slice(this.startIndex, this.endIndex);
    },

    hasNextPage() {
      return this.filteredTickers.length > this.endIndex;
    },

    filteredExamples() {
      return this.examples.slice(0, 5);
    },

    isAlreadyAdded() {
      return this.tickers.find(
        (ticker) => ticker.name === this.ticker && ticker.name !== ""
      );
    },
  },

  mounted() {
    window.addEventListener("resize", this.calculateGraphMaxElements);
  },
  methods: {
    formatPrice(price) {
      if (price === "-") {
        return price;
      }
      return price > 1 ? price.toFixed(2) : price.toPrecision(2);
    },

    async getCryptocurrencies() {
      const cryptocurrencies = await loadCryptocurrencies(),
        arrayOfValues = Object.values(cryptocurrencies["Data"]);

      arrayOfValues.forEach((a) => {
        this.fullNameOfExamples.push(a.FullName);
        this.examples.push(a.Symbol);
      });
    },
    updateTicker(tickerName, price) {
      this.tickers
        .filter((ticker) => ticker.name === tickerName)
        .forEach((ticker) => {
          if (ticker === this.selectedTicker) {
            while (this.graph.length > this.maxGraphElements) {
              this.graph.shift();
            }
            this.graph.push(price);
          }
          ticker.price = price;
        });
    },

    select(ticket) {
      if (ticket.price === "-") {
        return;
      }
      this.selectedTicker = ticket;
    },

    add(ticker) {
      const currentTicker = {
        name: ticker.toUpperCase(),
        price: "-",
      };

      if (this.isAlreadyAdded || ticker === "") {
        return;
      }

      this.tickers = [...this.tickers, currentTicker];
      this.filter = "";

      subscribeToTicker(currentTicker.name, (newPrice) =>
        this.updateTicker(currentTicker.name, newPrice)
      );
      localStorage.setItem("cryptonomicon-list", JSON.stringify(this.tickers));
    },

    handleDelete(tickerToRemove) {
      this.tickers = this.tickers.filter((t) => t !== tickerToRemove);
      if (this.selectedTicker === tickerToRemove) {
        this.selectedTicker = null;
      }
      unsubscribeFromTicker(tickerToRemove.name);
    },
  },

  watch: {
    async selectedTicker() {
      this.graph = [];
      this.$nextTick().then(this.calculateGraphMaxElements);
    },

    filter() {
      console.log("ok");
      this.page = 1;
    },

    ticker() {
      this.getCryptocurrencies();
      this.examples = this.examples.filter((example) =>
        example.toLowerCase().includes(this.ticker.toLowerCase())
      );
    },
  },
};
</script>
<style scoped>
.nav-button {
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
  focus:outline-none
  focus:ring-2
  focus:ring-offset-2
  focus:ring-gray-500;
}

.delete-button {
  @apply flex
  items-center
  justify-center
  font-medium
  w-full
  bg-gray-100
  px-4
  py-4
  sm:px-6
  text-gray-500
  hover:text-gray-600 hover:bg-gray-200 hover:opacity-20
  transition-all
  focus:outline-none;
}

.card {
  @apply bg-white
  overflow-hidden
  shadow
  rounded-lg
  border-purple-800 border-solid
  cursor-pointer;
}
</style>
