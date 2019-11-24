<template>

  <div>
    <!-- https://wireframe.cc/T4TPR6 -->
    <div class="max-w-sm rounded overflow-hidden shadow-lg">
      <div class="px-6 py-4">
        <div class="font-bold text-xl mb-2">The Coldest Sunset</div>
        <p class="text-gray-700 text-base">
          Lorem ipsum dolor sit amet, consectetur adipisicing elit. Voluptatibus quia, nulla! Maiores et perferendis eaque, exercitationem praesentium nihil.
        </p>
      </div>
      <div class="px-6 py-4">
        <span class="inline-block bg-gray-200 rounded-full px-3 py-1 text-sm font-semibold text-gray-700 mr-2">#photography</span>
        <span class="inline-block bg-gray-200 rounded-full px-3 py-1 text-sm font-semibold text-gray-700 mr-2">#travel</span>
        <span class="inline-block bg-gray-200 rounded-full px-3 py-1 text-sm font-semibold text-gray-700">#winter</span>
      </div>
    </div>


    <h1>Conversor de Moedas Cartão de Crédito</h1>
    <form>
      <select v-model="selectedCurrency" v-on:change="fetchPtax">
        <option v-for="currency in currencies" :key="currency.simbolo" :value="currency.simbolo">
          {{ currency.simbolo}} &mdash; {{ currency.nomeFormatado }}
        </option>
      </select>

      <label for="amount">Valor</label>
      <input type="text" id="amount" v-model="amount">

      <label for="spread">Spread</label>
      <input type="text" id="spread" v-model="spread">

      <label for="iof">IOF</label>
      <input type="text" id="iof" v-model="iof">

      Valor da Moeda: {{ finalCurrency }}
      Valor em Reais: {{ convertedAmount }}
      Valor Final: {{ finalAmount }}
    </form>
  </div>
</template>

<script>
import currencies from '../assets/currencies';
import responseOne from '../assets/response_one';

export default {
  data() {
    return {
      currencies,
      selectedCurrency: 'USD',
      amount: 0,
      ptaxCurrency: 0,
      spread: 0.04,
      iof: 0.0638,
    };
  },
  computed: {
    finalCurrency: function () {
      return this.ptaxCurrency + (this.spread * this.ptaxCurrency);
    },

    convertedAmount: function () {
      return this.amount * this.finalCurrency;
    },

    amountFee: function () {
      return this.convertedAmount * this.iof;
    },

    finalAmount: function () {
      return this.convertedAmount + this.amountFee;
    }
  },
  methods: {
    fetchPtax: async function () {
      try {
        // const response = await this.$axios.get(`https://olinda.bcb.gov.br/olinda/servico/PTAX/versao/v1/odata/CotacaoMoedaDia(moeda=@moeda,dataCotacao=@dataCotacao)?@moeda='${this.selectedCurrency}'&@dataCotacao='11-22-2019'&$top=100&$format=json`);
        // const exchangeRate = response.data.value;
        const exchangeRate = responseOne;
        const finalRate = exchangeRate[exchangeRate.length - 1];
        this.ptaxCurrency = finalRate.cotacaoVenda;
      } catch (error) {
        window.console.log(error);
      }

    }
  },
  mounted() {
    this.fetchPtax();
  }
}
</script>
