<template>
  <section>
    <div class="flex">
      <div class="max-w-xs">
        <label for="wallet" class="block text-sm font-medium text-gray-700"
        >Тикер {{ ticker }}</label>
        <div class="mt-1 relative rounded-md shadow-md">
          <input
              v-on:keydown.enter="add"
              v-model="ticker"
              type="text"
              name="wallet"
              id="wallet"
              class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
              placeholder="Например DOGE"
          />
        </div>
        <!--        <div v-if="ticker" class="flex bg-white shadow-md p-1 rounded-md shadow-md flex-wrap">
                            <span v-for="(example, idx) in filteredExamples" :key="idx + Math.random()" @click="ticker = example"
                                  class="inline-flex items-center px-2 m-1 rounded-md text-xs font-medium bg-gray-300 text-gray-800 cursor-pointer">
                              {{ example }}
                            </span>
                        </div>

                        <div v-if="tickers.find(t => t.name === ticker && t.name !== '')" class="text-sm text-red-600">Такой тикер уже
                          добавлен
                        </div>
                        -->


      </div>
    </div>
    <add-button @click="add"  :disabled="disabled"></add-button>

  </section>
</template>

<script>
import AddButton from './AddButton';


export default {
  name: "AddTicker",
  components: {AddButton},

  props: {
    disabled: {
      type: Boolean,
      required: false,
      default: false
    }
  },

  data() {
    return {
      ticker: '',
    }
  },

  methods: {
    add() {

      this.$emit('add-ticker', this.ticker);
      this.ticker = "";

    },
  }
}
</script>

<style scoped>

</style>