<template>
  <div class="relative w-full text-sm mb-8 cursor-pointer">
    <div class="w-full mr-1 p-1 leading-3 border-solid border-bl border-2">
      <button
        class="flex items-center justify-between w-full"
        :data-key="selectedCurrency.symbol"
        @click.prevent="toggleOptionsDisplay"
      >
        <span>{{ selectedCurrency.name }}</span
        ><img src="../assets/icons/dropdownIcon.png" alt="dropdown" />
      </button>
    </div>
    <div
      class="options absolute z-10 w-full h-28 flex flex-col p-1 mt-0 overflow-y-auto bg-white border-solid border-bl border-2 border-t-0"
      :class="optionsDisplay"
    >
      <ul>
        <li
          class="hover:bg-gray-300 w-full"
          v-for="(currency, index) in availableCurrencies"
          :key="index"
          :data-key="index"
          @click.prevent="updateSelectedCurrency"
        >
          {{ currency }}
        </li>
      </ul>
    </div>
  </div>
</template>
<script>
export default {
  name: 'DropdownMenu',
  props: ['selectedCurrency', 'availableCurrencies'],
  emits: ['updateCurrency', 'updateCurrencyPair'],
  data() {
    return {
      isOptionsHidden: true
    };
  },
  methods: {
    toggleOptionsDisplay() {
      this.isOptionsHidden = !this.isOptionsHidden;
    },
    updateSelectedCurrency(event) {
      this.isOptionsHidden = true;
      const name = event.target.textContent;
      const symbol = event.target.getAttribute('data-key');

      const currency = {
        name: name,
        symbol: symbol
      };

      this.$emit('updateCurrency', currency);
      this.$emit('updateCurrencyPair');
    }
  },
  computed: {
    optionsDisplay() {
      return this.isOptionsHidden ? 'hidden' : '';
    }
  }
};
</script>
