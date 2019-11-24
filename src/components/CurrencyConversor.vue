<template>

  <div>
    <!-- https://wireframe.cc/T4TPR6 -->
    <div class=" container mx-auto py-10">
      <div class="border m-6 rounded-lg  bg-white mx-auto max-w-sm shadow-lg rounded-lg overflow-hidden">
        <div class="sm:flex sm:items-center px-6 py-4">
          <img class="block h-16 sm:h-24 rounded-full mx-auto mb-4 sm:mb-0 sm:mr-4 sm:ml-0" src="https://api.adorable.io/avatars/196/abott@adorable.png" alt="">
          <div class="text-center sm:text-left sm:flex-grow">
            <div class="mb-4">
              <p class="text-xl leading-tight">Jane Doe</p>
              <p class="text-sm leading-tight text-grey-dark">Software Developer at SpongeBob LLC.</p>
            </div>
            <div class="flex flex-wrap">
              <button class=" text-xs font-semibold rounded-full px-4 py-1 mx-3  leading-normal bg-white border border-blue text-blue hover:bg-blue hover:text-white">Call</button>
              <button class="  text-xs font-semibold rounded-full px-4 py-1 leading-normal bg-white border border-purple text-purple hover:bg-purple hover:text-white">Message</button>
            </div>
          </div>
        </div>
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
