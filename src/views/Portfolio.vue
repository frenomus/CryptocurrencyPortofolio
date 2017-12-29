<template>
  <div class="animated fadeIn">
    <b-row class="portfolio-control-row">
      <b-col
        md="6"
        class="">
        <b-form-group
          horizontal
          label="Currency"
          class="mb-0">
          <b-form-select
            :options="localCurrencyOptions"
            v-model="localCurrencyId"/>
        </b-form-group>
      </b-col>
      <b-col
        md="6"
        class="">
        <div class="pull-right">
          <b-button
            variant="primary"
            v-b-modal.addPortfolioItem>
            <i class="fa fa-plus"/>&nbsp; Add item
          </b-button>
        </div>
      </b-col>
    </b-row>
    <b-row>
      <b-col
        cols="6"
        v-if="localCurrencyIdForValues !== 'usd'">
        <b-card :no-body="true">
          <b-card-body class="p-0 clearfix">
            <i class="bg-secondary p-4 px-5 font-2xl mr-3 float-left">USD</i>
            <div class="h5 text-secondary mb-0 pt-3">{{ portfolioValueUSD }}</div>
            <div class="text-muted text-uppercase font-weight-bold font-xs">
              Portfolio value
            </div>
          </b-card-body>
        </b-card>
      </b-col>
      <b-col cols="6">
        <b-card :no-body="true">
          <b-card-body class="p-0 clearfix">
            <i class="bg-primary text-uppercase p-4 px-5 font-2xl mr-3 float-left">
              {{ localCurrencyIdForValues }}
            </i>
            <div class="h5 text-primary mb-0 pt-3">{{ portfolioValueLocalCurrency }}</div>
            <div class="text-muted text-uppercase font-weight-bold font-xs">
              Portfolio value
            </div>
          </b-card-body>
        </b-card>
      </b-col>
    </b-row>
    <portfolio-cards
      :items="portfolioData"
      :local-currency-id="localCurrencyId"
      :currency-data="currencyData"
    />
    <b-modal
      id="addPortfolioItem"
      title="Add portfolio entry"
      @ok="addPortfolioEntry">
      <portfolio-add-item-form
        :crypto-currency-list="currencyList"
        :form="portfolioItemToAdd"
        :portfolio="portfolioData"/>
    </b-modal>
  </div>
</template>

<script>
import PortfolioCards from '../components/PortfolioCards.vue';
import PortfolioAddItemForm from '../components/PortfolioAddItemForm.vue';
import axios from 'axios';

function buildUrl(currency) {
  return `https://api.coinmarketcap.com/v1/ticker/?limit=0&convert=${currency}`;
}

function calcPortfolioValue(propertyName, portfolioData, currencyData) {
  let sum = 0;

  Object.keys(portfolioData).forEach(_key => {
    if (currencyData[_key] === undefined) {
      return;
    }

    sum += portfolioData[_key].value * (currencyData[_key][propertyName] || 0);
  });

  return Number(Math.round(sum + 'e2') + 'e-2');
}

export default {
  name: 'Portfolio',
  components: { PortfolioCards, PortfolioAddItemForm },
  data: () => {
    return {
      portfolioItemToAdd: {
        id: '',
        value: ''
      },
      currencyList: [],
      portfolioData: {
        bitcoin: { id: 'bitcoin', value: 0.75 },
        ethereum: { id: 'ethereum', value: 1.5 }
      },
      currencyData: {},
      localCurrencyId: 'usd',
      localCurrencyIdForValues: 'usd',
      localCurrencyOptions: [
        { 'value': 'aud', 'text': 'AUD' },
        { 'value': 'brl', 'text': 'BRL' },
        { 'value': 'cad', 'text': 'CAD' },
        { 'value': 'chf', 'text': 'CHF' },
        { 'value': 'clp', 'text': 'CLP' },
        { 'value': 'cny', 'text': 'CNY' },
        { 'value': 'czk', 'text': 'CZK' },
        { 'value': 'dkk', 'text': 'DKK' },
        { 'value': 'eur', 'text': 'EUR' },
        { 'value': 'gbp', 'text': 'GBP' },
        { 'value': 'hkd', 'text': 'HKD' },
        { 'value': 'huf', 'text': 'HUF' },
        { 'value': 'idr', 'text': 'IDR' },
        { 'value': 'ils', 'text': 'ILS' },
        { 'value': 'inr', 'text': 'INR' },
        { 'value': 'jpy', 'text': 'JPY' },
        { 'value': 'krw', 'text': 'KRW' },
        { 'value': 'mxn', 'text': 'MXN' },
        { 'value': 'myr', 'text': 'MYR' },
        { 'value': 'nok', 'text': 'NOK' },
        { 'value': 'nzd', 'text': 'NZD' },
        { 'value': 'php', 'text': 'PHP' },
        { 'value': 'pkr', 'text': 'PKR' },
        { 'value': 'pln', 'text': 'PLN' },
        { 'value': 'rub', 'text': 'RUB' },
        { 'value': 'sek', 'text': 'SEK' },
        { 'value': 'sgd', 'text': 'SGD' },
        { 'value': 'thb', 'text': 'THB' },
        { 'value': 'try', 'text': 'TRY' },
        { 'value': 'twd', 'text': 'TWD' },
        { 'value': 'usd', 'text': 'USD' },
        { 'value': 'zar', 'text': 'ZAR' }]
    };
  },
  computed: {
    portfolioValueUSD() {
      return calcPortfolioValue('price_usd', this.portfolioData, this.currencyData);
    },
    portfolioValueLocalCurrency() {
      return calcPortfolioValue('localPrice', this.portfolioData, this.currencyData);
    }
  },
  watch: {
    localCurrencyId: function localCurrencyId() {
      this.getCurrencyData();
    }
  },
  created: function create() {
    this.getCurrencyData();
  },
  methods: {
    getCurrencyData() {
      const currencyId = this.localCurrencyId;
      const url = buildUrl(currencyId);

      axios.get(url).then(response => {
        if (!Array.isArray(response.data)) {
          throw new Error('Object returned was not an array');
        }

        const _currencyData = {};
        const _currencyList = [];

        response.data.forEach(_currency => {
          _currencyData[_currency.id] = _currency;
          _currencyData[_currency.id].localSymbol = currencyId;
          _currencyData[_currency.id].localPrice =
            _currency[`price_${this.localCurrencyId}`];
          _currencyList.push({ text: _currency.name, value: _currency.id });
        });

        this.currencyData = _currencyData;
        this.currencyList = _currencyList;
        this.localCurrencyIdForValues = currencyId;
      }).catch(e => { this.errors.push(e); });
    },
    addPortfolioEntry(bvEvt) {
      if (this.portfolioItemToAdd.id === '' ||
          this.portfolioItemToAdd.value === '') {
        //Trying to add an entry with no currency name defined or a missing value
        bvEvt.preventDefault();

        return false;
      }

      this.$set(this.portfolioData, this.portfolioItemToAdd.id, {
        id: this.portfolioItemToAdd.id,
        /** Force to be a number **/
        value: this.portfolioItemToAdd.value * 1
      });

      this.portfolioItemToAdd.id = '';
      this.portfolioItemToAdd.value = '';
    }
  }
};
</script>

<style>
  .portfolio-control-row {
    margin-bottom: 1em;
  }
</style>