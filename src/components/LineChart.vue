<template>
  <div class="relative w-full rounded-2xl shadow-2xl">
    <div class="flex justify-between">
      <div class="mt-1 ml-1">
        <div class="flex items-center justify-between">
          <div class="currency-flag currency-flag-lg mr-3 rounded-[50%]" :class="updateBaseCurrencyFlag"></div>
          <div class="currency-flag currency-flag-lg mr-3 rounded-[50%]" :class="updateQuoteCurrencyFlag"></div>
          <a
            href="https://www.forex.com"
            target="_blank"
            class="no-underline text-gray-600 text-xs bg-gray-100 p-1 rounded-2xl"
            >Forex.com</a
          >
        </div>
        <div class="mt-1">
          <span class="font-extrabold text-sm md:text-base">{{ selectedCurrencyPairSymbol }}</span>
        </div>
      </div>
      <div class="flex flex-col gap-2 w-4/5 sm:w-2/5 mt-1 text-[8px] md:text-sm">
        <div class="flex justify-center mt-1 text-sm md:text-lg">
          <span id="liveRate"></span>
        </div>
        <div class="flex justify-center text-[9px] md:text-sm">
          <span :class="differenceStyle" class="asd mr-1">{{ difference }}</span>
          <span :class="differenceStyle">{{ percentageRate }}</span>
        </div>
      </div>
    </div>
    <div class="relative w-full h-auto mt-5 mb-2">
      <myChart v-if="isChartDataLoaded" :options="chartOptions" :data="chartData" />
    </div>
    <div class="flex w-3/4 justify-around m-auto">
      <button
        class="tabs text-xs p-1 hover:bg-sky-100 rounded-lg cursor-pointer"
        v-for="tab in chartTabs"
        :key="tab"
        :data-key="tab"
        :class="{
          'font-bold bg-sky-100': tab === selectedTab
        }"
        @click.prevent="updateSelectedTab"
      >
        {{ tab }}
      </button>
    </div>
  </div>
</template>

<script>
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
} from 'chart.js';
import { Line as myChart } from 'vue-chartjs';

ChartJS.register(CategoryScale, LinearScale, PointElement, LineElement, Title, Tooltip, Legend);

export default {
  name: 'LineChart',
  components: { myChart },
  props: ['selectedCurrencies', 'selectedCurrencyPairSymbol', 'selectedCurrencyPair', 'isCurrencyPairChanged'],
  data() {
    return {
      chartData: {
        labels: [],
        datasets: [{ data: [], borderColor: '#33CC33', label: `${this.selectedCurrencyPairSymbol}` }]
      },
      chartOptions: {
        responsive: true,
        plugins: {
          legend: {
            display: false
          }
        },
        elements: {
          line: {
            fill: true
          }
        },
        scales: {
          x: {
            grid: {
              display: false
            }
          },
          y: {
            grid: {
              display: false
            }
          }
        }
      },
      //The tabs to show the data of the requested time interval.
      chartTabs: ['15M', '1H', '1D', '1W', '1M'],
      selectedTab: '15M',
      difference: '',
      differencePercentage: '',
      //A boolean to be used to check if the data is fetched.
      isChartDataLoaded: false
    };
  },
  methods: {
    updateSelectedTab(event) {
      //Update selected tab, fetch data then plot the chart.
      this.selectedTab = event.target.getAttribute('data-key');
      this.plotChart(this.selectedCurrencyPair, import.meta.env.VITE_API_KEY, this.setTimeInterval());
    },
    startDate(value) {
      //Gets the current time as milisecond then substracts value * one minute from the current time
      const startDate = new Date(new Date().getTime() - value * 60000);

      //To send the correct time format to the API each time piece must have two digits.
      //If it is not, add zero to the beginning
      const startDateYear = startDate.getFullYear().toString().padStart(2, '0');

      //Since the months are 0-11, add 1 to get the right one
      const startDateMonth = (startDate.getUTCMonth() + 1).toString().padStart(2, '0');
      const startDateDay = startDate.getUTCDate().toString().padStart(2, '0');
      const startDateHours = startDate.getUTCHours().toString().padStart(2, '0');
      const startDateMinutes = startDate.getUTCMinutes().toString().padStart(2, '0');

      //Format the start date per selected time interval
      switch (this.selectedTab) {
        case '15M':
        case '1H':
        case '1D':
          return `start_date=${startDateYear}-${startDateMonth}-${startDateDay}-${startDateHours}:${startDateMinutes}`;
        case '1W':
          return `start_date=${startDateYear}-${startDateMonth}-${startDateDay}`;
        case '1M':
          return `start_date=${startDateYear}-${startDateMonth}-${startDateDay}`;
      }
    },
    endDate() {
      const currentTime = new Date(new Date().getTime());

      const currentYear = currentTime.getFullYear().toString().padStart(2, '0');
      const currentMonth = (currentTime.getUTCMonth() + 1).toString().padStart(2, '0');
      const currentDay = currentTime.getUTCDate().toString().padStart(2, '0');
      const currentHours = currentTime.getUTCHours().toString().padStart(2, '0');
      const currentMinutes = currentTime.getUTCMinutes().toString().padStart(2, '0');

      //Format the end date per selected time interval
      switch (this.selectedTab) {
        case '15M':
        case '1H':
        case '1D':
          return `&end_date=${currentYear}-${currentMonth}-${currentDay}-${currentHours}:${currentMinutes}`;
        case '1W':
          return `&end_date=${currentYear}-${currentMonth}-${currentDay}`;
        case '1M':
          return `&end_date=${currentYear}-${currentMonth}-${currentDay}`;
      }
    },
    setTimeInterval() {
      //Set the completed time interval to be insterted to the fetch API url
      switch (this.selectedTab) {
        case '15M':
          return `${this.startDate(15)}${this.endDate()}&format=records&interval=minute&period=5`;
        case '1H':
          return `${this.startDate(60)}${this.endDate()}&format=records&interval=minute&period=10`;
        case '1D':
          return `${this.startDate(1440)}${this.endDate()}&format=records&interval=hourly&period=4`;
        case '1W':
          return `${this.startDate(10080)}${this.endDate()}&format=records&interval=hourly&period=24`;
        case '1M':
          return `${this.startDate(43200)}${this.endDate()}&format=records&interval=hourly&period=24`;
      }
    },
    async plotChart(currencyPair, apiKey, timeInterval) {
      //The chart is being rendered per this boolean.
      //Do not render the chart till the new datas have fetched
      this.isChartDataLoaded = false;

      //Fetch Data
      try {
        await fetch(
          `https://marketdata.tradermade.com/api/v1/timeseries?currency=${currencyPair}&api_key=${apiKey}&${timeInterval}`,
          {
            method: 'GET',
            redirect: 'follow'
          }
        )
          .then((response) => response.text())
          .then((result) => JSON.parse(result))
          .then((result) => {
            //Exract the necessary dataset.
            const labels = result.quotes.map((data) => data.date);
            const data = result.quotes.map((data) => data.close);

            //Calculate the difference and the difference percent.
            const difference = Number((data[data.length - 1] - data[0]).toString().slice(0, 9));
            const differencePercentage = ((data[data.length - 1] - data[0]) / data[0]) * 100;

            //Update the component datas that the chart is being binded.
            this.chartData.labels = labels;
            this.chartData.datasets[0].data = data;
            this.difference = difference;
            this.differencePercentage = differencePercentage;
          });

        //Render the chart with the new dataset.
        this.isChartDataLoaded = true;
      } catch (error) {
        alert('An error occured. Error', error.message);
      }
    },
    live(value) {
      //Define socket.
      let socket = new WebSocket('wss://marketdata.tradermade.com/feedadv');

      //Send the userkey and selected currency pair.
      socket.onopen = function (e) {
        socket.send(JSON.stringify({ userKey: 'wsbMsDUmsJ_JfyEiat4w', symbol: value }));
      };

      //Get the live FX rate.
      socket.onmessage = function (event) {
        document.getElementById('liveRate').textContent = JSON.parse(event.data).mid;
      };

      return socket;
    }
  },
  computed: {
    //Update the flags when user select another currency
    updateBaseCurrencyFlag() {
      return `currency-flag-${this.selectedCurrencies.baseCurrency.symbol.toLowerCase()}`;
    },
    updateQuoteCurrencyFlag() {
      return `currency-flag-${this.selectedCurrencies.quoteCurrency.symbol.toLowerCase()}`;
    },
    differenceStyle() {
      return this.difference < 0 ? 'text-red-600' : 'text-green-600';
    },
    percentageRate() {
      return `(${this.difference.toString().slice(0, 9)}%)`;
    }
  },
  async mounted() {
    //When the component is being mounted, get the live rate and plot the chart.
    this.live(this.selectedCurrencyPair);
    this.plotChart(this.selectedCurrencyPair, import.meta.env.VITE_API_KEY, this.setTimeInterval());
  },
  //Watch the changes on the currencyPair.
  //If currencyPair changes, re-plot the chart, then restart the socket and get the live rate per selected pair.
  watch: {
    selectedCurrencyPair() {
      this.plotChart(this.selectedCurrencyPair, import.meta.env.VITE_API_KEY, this.setTimeInterval());
      this.live(this.selectedCurrencyPair).close();
      this.live(this.selectedCurrencyPair);
    }
  }
};
</script>
