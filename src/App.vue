<template>
  <section class="mt-[15vh] h-[50vh] flex flex-col items-center md:flex-row">
    <div class="w-44 md:ml-[9%] mr-[4%] border-solid">
      <DropdownMenu
        :availableCurrencies="availableCurrencies"
        :selectedCurrency="selectedCurrencies.baseCurrency"
        @updateCurrency="updateCurrencyBase"
        @updateCurrencyPair="updateCurrencyPair"
      />
      <DropdownMenu
        :availableCurrencies="availableCurrencies"
        :selectedCurrency="selectedCurrencies.quoteCurrency"
        @updateCurrency="updateCurrencyQuote"
        @updateCurrencyPair="updateCurrencyPair"
      />
    </div>
    <div class="w-[95%] md:w-3/6 md:mr-[9%]">
      <LineChart
        :selectedCurrencies="selectedCurrencies"
        :selectedCurrencyPair="selectedCurrencyPair"
        :selectedCurrencyPairSymbol="selectedCurrencyPairSymbol"
      />
    </div>
  </section>
</template>

<script>
import DropdownMenu from './components/DropDownMenu.vue';
import LineChart from './components/LineChart.vue';

export default {
  name: 'App',
  components: {
    DropdownMenu,
    LineChart
  },
  data() {
    return {
      //Stores the live currencies listed on the TraderMade
      availableCurrencies: {},
      selectedCurrencies: {
        baseCurrency: {
          name: 'Euro',
          symbol: 'EUR'
        },
        quoteCurrency: {
          name: 'Us Dollar',
          symbol: 'USD'
        }
      },
      selectedCurrencyPair: 'EURUSD',
      selectedCurrencyPairSymbol: 'EUR/USD'
    };
  },
  methods: {
    //Emits that are being used to update the parent component's data.
    updateCurrencyBase(currency) {
      this.selectedCurrencies.baseCurrency = currency;
    },
    updateCurrencyQuote(currency) {
      this.selectedCurrencies.quoteCurrency = currency;
    },
    updateCurrencyPair() {
      this.selectedCurrencyPair =
        this.selectedCurrencies.baseCurrency.symbol + this.selectedCurrencies.quoteCurrency.symbol;
      this.selectedCurrencyPairSymbol = `${this.selectedCurrencies.baseCurrency.symbol}/${this.selectedCurrencies.quoteCurrency.symbol}`;
    },
    //Fetch the live currencies listed on the TraderMade.
    getLiveCurrencyList() {
      fetch(`https://marketdata.tradermade.com/api/v1/live_currencies_list?api_key=${import.meta.env.VITE_API_KEY}`, {
        method: 'GET',
        redirect: 'follow'
      })
        .then((response) => response.text())
        .then((result) => JSON.parse(result))
        .then((result) => (this.availableCurrencies = result.available_currencies))
        .catch((error) => alert('An error occured. Error', error.message));
    }
  },
  created() {
    //Fetch the live currencies while the component is being created.
    this.getLiveCurrencyList();
  }
};
</script>
