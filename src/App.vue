<template>
  <div class="container">

    <add-ticker @add-ticker="add" :disabled="tooManyTickersAdded"/>
    <div>
      <button
          v-if="page > 1"
          @click="page = page - 1"
          class=" my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500">
        Prev
      </button>
      <button
          v-if="hasNextPage"
          @click="page = page + 1"
          class="m-4 my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500">
        Next
      </button>
      <div class="inline">
        Filter: <input v-model="filter" type="text">
      </div>

    </div>

    {{ page }}
    <template v-if="tickers.length">
      <hr class="w-full border-t border-gray-600 my-4"/>

      <dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
        <div v-for="t in paginatedTickers" :key="Math.random() + t.price" @click="select(t)"
             :class="{'border-2': selectedTicker === t}"
             class="bg-white overflow-hidden shadow rounded-lg border-purple-800  border-solid cursor-pointer ">
          <div class="px-4 py-5 sm:p-6 text-center" v-bind:class="{'bg-red-200': t.price === '-' }">
            <dt class="text-sm font-medium text-gray-500 truncate">
              {{ t.name }} - USD
            </dt>
            <dd class="mt-1 text-3xl font-semibold text-gray-900">
              {{ formattedPrice(t.price) }}
            </dd>
          </div>
          <div class="w-full border-t border-gray-200"></div>

          <button @click.stop="handleDelete(t)"
                  class="flex items-center justify-center font-medium w-full bg-gray-100 px-4 py-4 sm:px-6 text-md text-gray-500 hover:text-gray-600 hover:bg-gray-200 hover:opacity-20 transition-all focus:outline-none">
            <svg
                class="h-5 w-5"
                xmlns="http://www.w3.org/2000/svg"
                viewBox="0 0 20 20"
                fill="#718096"
                aria-hidden="true"
            >
              <path
                  fill-rule="evenodd"
                  d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
                  clip-rule="evenodd"
              ></path>
            </svg>
            Удалить
          </button>
        </div>

      </dl>
      <hr class="w-full border-t border-gray-600 my-4 bg-gradient-to-b"/>
    </template>
    <ticker-graph v-if="selectedTicker" :ticker="selectedTicker" :graph="graph" @hide-graph="selectedTicker = null"/>

  </div>

</template>

<script>

import {subscribeToTicker, unsubscribeFromTicker} from "./api";
import {loadCryptocurrencies} from "@/api";

import AddTicker from "@/components/AddTicker";
import TickerGraph from "@/components/TickerGraph";


export default {
  name: 'App',
  components: {TickerGraph, AddTicker},
  data() {
    return {

      tickers: [],
      selectedTicker: null,
      graph: [],
      examples: [],
      page: 1,
      filter: '',
      fullNameOfExamples: [],


    };
  },

  created() {
    this.getCryptocurrencies();

    const windowData = Object.fromEntries(new URL(window.location).searchParams.entries());

    if (windowData.filter) {
      this.filter = windowData.filter;
    }

    if (windowData.page) {
      this.page = windowData.page;
    }

    const tickersData = localStorage.getItem('cryptonomicon-list');

    if (tickersData) {
      this.tickers = JSON.parse(tickersData);
      this.tickers.forEach(ticker => {
        subscribeToTicker(ticker.name, newPrice =>
            this.updateTicker(ticker.name, newPrice)
        );
      });
    }


    setInterval(this.updateTickers, 3000);

  },

  computed: {

    tooManyTickersAdded() {
      return this.tickers.length > 10;
    },

    pageStateOptions() {
      return {
        filter: this.filter,
        page: this.page,
      }

    },

    startIndex() {
      return (this.page - 1) * 6;
    },

    endIndex() {
      return this.page * 6;
    },

    filteredTickers() {
      return this.tickers.filter(ticker => ticker.name.includes(this.filter));
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
      return this.tickers.find(t => t.name === this.ticker && t.name !== '');
    },


  },

  mounted() {
    window.addEventListener('resize', this.calculateGraphMaxElements);
  },

  beforeMount() {
    window.removeEventListener('resize', this.calculateGraphMaxElements);
  },

  methods: {


    formattedPrice(price) {
      if (price === "-") {
        return price;
      }
      return price > 1 ? price.toFixed(2) : price.toPrecision(2);
    },


    async getCryptocurrencies() {

      const cryptocurrencies = await loadCryptocurrencies(),
          arrayOfValues = Object.values(cryptocurrencies['Data']);


      arrayOfValues.forEach((a) => {
        this.fullNameOfExamples.push(a.FullName);
        this.examples.push(a.Symbol)
      });

    },
    updateTicker(tickerName, price) {
      this.tickers
          .filter(t => t.name === tickerName)
          .forEach(t => {
            if (t === this.selectedTicker) {
              while (this.graph.length > this.maxGraphElements) {
                this.graph.shift();
              }
              this.graph.push(price);

            }
            t.price = price;
          });
    },

    select(ticket) {

      if (ticket.price === '-') {
        return;
      }
      this.selectedTicker = ticket;

    },

    add(ticker) {
      const currentTicker = {
        name: ticker,
        price: "-"
      };

      if (this.isAlreadyAdded || ticker === '') {
        return
      }


      this.tickers = [...this.tickers, currentTicker];
      this.filter = "";

      subscribeToTicker(currentTicker.name, newPrice =>
          this.updateTicker(currentTicker.name, newPrice)
      );
    },


    handleDelete(tickerToRemove) {
      this.tickers = this.tickers.filter(t => t !== tickerToRemove);
      if (this.selectedTicker === tickerToRemove) {
        this.selectedTicker = null;
      }
      unsubscribeFromTicker(tickerToRemove.name);
    },
  },

  watch: {

    tickers() {
      localStorage.setItem('cryptonomicon-list', JSON.stringify(this.tickers));
    },
    async selectedTicker() {
      this.graph = [];
      this.$nextTick().then(this.calculateGraphMaxElements);
    },

    paginatedTickers() {
      if (this.paginatedTickers.length === 0 && this.page > 1) {
        this.page -= 1;
      }
    },

    pageStateOptions(value) {
      const title = document.title;
      history.pushState(null, title, `${window.location.pathname}?filter=${value.filter}&page=${value.page}`)
    },

    filter() {
      this.page = 1;
    },

    ticker() {
      this.getCryptocurrencies();
      this.examples = this.examples.filter(x => x.toLowerCase().includes(this.ticker.toLowerCase()));
    }

  },
}
</script>



