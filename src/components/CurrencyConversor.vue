<template>
    <div class="max-w-2xl mt-10 mx-auto px-4">
        <!-- https://wireframe.cc/T4TPR6 -->
        <div class="rounded bg-white shadow-lg">
            <div class="px-6 py-4">
                <h1 class="text-3xl font-normal mb-6 leading-tight">Conversor de Moedas Cartão de Crédito</h1>

                <form autocomplete="off">
                    <div class="flex flex-wrap -mx-3 mb-6">
                        <!-- SELECT -->
                        <div class="w-full px-3 mb-6">
                            <label class="block uppercase tracking-wide text-gray-700 text-xs font-bold mb-2"
                                   for="original-currency">
                                Moeda de Origem
                            </label>
                            <div class="relative">
                                <select v-model="selectedCurrency" v-on:change="fetchPtax"
                                        class="block appearance-none w-full bg-gray-200 border border-gray-200 text-gray-700 py-3 px-4 pr-8 rounded leading-tight focus:outline-none focus:bg-white focus:border-gray-500"
                                        id="original-currency">
                                    <option v-for="currency in currencies" :key="currency.simbolo"
                                            :value="currency.simbolo">
                                        {{ currency.simbolo}} &mdash; {{ currency.nomeFormatado }}
                                    </option>
                                </select>
                                <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-700">
                                    <svg class="fill-current h-4 w-4" xmlns="http://www.w3.org/2000/svg"
                                         viewBox="0 0 20 20">
                                        <path d="M9.293 12.95l.707.707L15.657 8l-1.414-1.414L10 10.828 5.757 6.586 4.343 8z"/>
                                    </svg>
                                </div>
                            </div>
                        </div>
                        <!-- SELECT END -->

                        <!-- SPREAD -->
                        <div class="w-full md:w-1/3 px-3 mb-6 md:mb-0">
                            <label class="block uppercase tracking-wide text-gray-700 text-xs font-bold mb-2"
                                   for="spread">
                                Spread
                            </label>
                            <money
                                    v-model="spread"
                                    v-bind="percentage"
                                    class="appearance-none block w-full bg-gray-200 text-gray-700 border border-gray-200 rounded py-3 px-4 mb-3 leading-tight focus:outline-none focus:bg-white focus:border-gray-500"
                                    id="spread"
                                    type="tel"/>
                            <p class="text-gray-600 text-xs italic">Percentual adicionado ao valor da moeda para
                                manutenção de custos operacionais.</p>
                        </div>
                        <!-- SPREAD END -->

                        <!-- IOF -->
                        <div class="w-full md:w-1/3 px-3 mb-6 md:mb-0">
                            <label class="block uppercase tracking-wide text-gray-700 text-xs font-bold mb-2" for="iof">
                                IOF
                            </label>
                            <money v-model="iof"
                                   v-bind="percentage"
                                   class="appearance-none block w-full bg-gray-200 text-gray-700 border border-gray-200 rounded py-3 px-4 mb-3 leading-tight focus:outline-none focus:bg-white focus:border-gray-500"
                                   id="iof"
                                   type="tel"/>
                            <p class="text-gray-600 text-xs italic">Valor do IOF praticado. Geralmente é de 6,38%.</p>
                        </div>
                        <!-- IOF END -->
                    </div>

                    <hr>

                    <div class="flex flex-wrap flex-col mt-6 -mx-3 mb-2 justify-center items-center">
                        <!-- AMOUNT -->
                        <div class="px-3 mb-6">
                            <div class="text-2xl font-mono">
                                <label for="amount">{{ selectedCurrency}}</label>
                                <money id="amount"
                                       type="tel"
                                       v-model="amount"
                                       v-bind="money"
                                       class="ml-3"
                                       title="Digite o valor total da compra na moeda estrangeira."
                                       v-bind:style="{ width: defaultWidth }"
                                       v-on:input="resizeInputToFit"
                                       ref="moneyReference"/>
                            </div>
                        </div>
                        <!-- AMOUNT END -->

                        <div class="px-3 mb-6 text-center text-gray-600">
                            <p class="font-mono">{{ selectedCurrency }} 1,00 &rarr; BRL {{
                                finalCurrency.toFixed(2).replace('.', ',') }}</p>
                            <p class="text-xs italic">Cotação PTAX do dia anterior acrescido do spread.</p>
                        </div>

                        <div class="px-3 mb-6 text-center text-gray-600">
                            <p class="font-mono">
                                {{ selectedCurrency }} {{ amount.toFixed(2).replace('.', ',') }} &rarr; BRL {{
                                convertedAmount.toFixed(2).replace('.', ',') }}
                            </p>
                            <p class="text-xs italic">Conversão do valor da compra utilizando o valor da moeda
                                praticado.</p>
                        </div>

                        <div class="px-3 mb-6 text-center">
                            <p class="font-mono text-green-600">
                                + BRL {{ amountFee.toFixed(2).replace('.', ',') }}
                            </p>
                            <p class="text-gray-600 text-xs italic">Acréscimo do Imposto sobre Operações Financeiras
                                (IOF).</p>
                        </div>

                        <div class="px-3 mb-6 text-center">
                            <p class="text-2xl font-mono">
                                BRL {{ finalAmount.toFixed(2).replace('.', ',') }}
                            </p>
                            <p class="uppercase text-xs text-gray-500">Total</p>
                        </div>

                    </div>
                </form>
            </div>
        </div>
    </div>
</template>

<script>
    import currencies from '../assets/currencies';
    import {Money} from 'v-money'

    export default {
        components: {Money},

        data() {
            return {
                currencies,
                selectedCurrency: 'USD',
                amount: 0,
                money: {
                    decimal: ',',
                    thousands: '.',
                    prefix: '',
                    suffix: '',
                    precision: 2,
                    masked: false,
                },
                percentage: {
                    decimal: ',',
                    thousands: '.',
                    prefix: '',
                    suffix: ' %',
                    precision: 2,
                    masked: false,
                },
                ptaxCurrency: 0,
                spread: 4,
                iof: 6.38,
                defaultWidth: '5ch',
                isMounted: false,
            };
        },

        computed: {
            quoteDate: function () {
                const now = new Date();
                const day = now.getDay();
                let subDays = 1;
                if (day === 0) {
                    // Sub two days from sunday.
                    subDays = 2;
                }
                if (day === 1) {
                    // Sub three days from monday.
                    subDays = 3;
                }
                now.setDate(now.getDate() - subDays);

                return `${now.getMonth() + 1}-${now.getDate()}-${now.getFullYear()}`;
            },

            finalCurrency: function () {
                return this.ptaxCurrency + ((this.spread / 100) * this.ptaxCurrency);
            },

            convertedAmount: function () {
                return this.amount * this.finalCurrency;
            },

            amountFee: function () {
                return this.convertedAmount * (this.iof / 100);
            },

            finalAmount: function () {
                return this.convertedAmount + this.amountFee;
            }
        },
        methods: {
            fetchPtax: async function () {
                try {
                    const response = await this.$axios.get(`https://olinda.bcb.gov.br/olinda/servico/PTAX/versao/v1/odata/CotacaoMoedaDia(moeda=@moeda,dataCotacao=@dataCotacao)?@moeda='${this.selectedCurrency}'&@dataCotacao='${this.quoteDate}'&$top=100&$format=json`);
                    const exchangeRate = response.data.value;
                    const finalRate = exchangeRate[exchangeRate.length - 1];
                    this.ptaxCurrency = finalRate.cotacaoVenda;
                } catch (error) {
                    window.console.log(error);
                }

            },

            resizeInputToFit: function () {
                if (this.isMounted) {
                    let moneyComponent = this.$refs.moneyReference;
                    this.defaultWidth = (moneyComponent.formattedValue.length + 1) + 'ch';
                }
            }
        },
        mounted() {
            this.fetchPtax();
            this.$nextTick(function () {
                // Code that will run only after the entire view has been rendered
                this.isMounted = true;
            });
        }
    }
</script>
