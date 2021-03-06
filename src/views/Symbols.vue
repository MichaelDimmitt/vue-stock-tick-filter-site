<template>
  <div>
    <div class="primary-heading-con">
      <div class="heading">
        <div class="title">Symbols/ Tickers</div>
        <input v-model="searchText" placeholder="Search Tick/Company" />&nbsp;
        <button @click="searchButton">
          <i class="fa fa-search" aria-hidden="true"></i>
        </button> &nbsp;
        <button v-if="excludeTickers" @click="resetTickers">Restore Ignored Tickers</button>
        <br />
        <br />
        <h2>Filter Logic</h2>
        <div class="is-flex">
          <ul class="centerx">
            <li><input type="checkbox" id="symbol" value="symbol" v-model="fieldsToFilter" /> Symbol</li>
            <li><input type="checkbox" id="open" value="open" v-model="fieldsToFilter" /> Open</li>
            <li><input type="checkbox" id="close" value="close" v-model="fieldsToFilter" /> Close</li>
            <li><input type="checkbox" id="primaryExchange" value="primaryExchange" v-model="fieldsToFilter" /> Primary Exchange</li>
          </ul>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
          <div style="display: flex; flex-direction: column; justify-content: space-between;">
            <button @click="ascend">Ascending</button>
            <button @click="descend">Descending</button>
            <button @click="removeSort">Remove Sort</button>
          </div>
        </div>
      </div>
    </div>

    <div class="page-content-con">
      <loading v-if="loading"></loading>
      <div v-else style="display: flex;flex-wrap: wrap;padding-left: 10px;padding-right: 10px;">
        <div
          v-for="company in filteredCompanies"
          :key="company.symbol"
          class="company"
          style="padding-right: 30px;width: 374.45px;"
        >
          <h5 class="heading is-size-5">
            <a
              v-bind:href="'https://www.google.com/search?q='+company.companyName+' stock'"
            >{{company.symbol}}</a> :
            <small class="is-size-7">{{company.companyName}}</small>
          </h5>
          <div>
            Open
            <money :value="company.open"></money>
          </div>
          <div>
            Close
            <money :value="company.close"></money>
          </div>
          <timestamp :value="company.openTime"></timestamp>-
          <timestamp :value="company.closeTime"></timestamp>
          <button v-bind:value="company.symbol" @click="exclude">exclude</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import API from "../api/IEX";
import { sort, search, filterCompanies, reject } from "../lib/apiFilter";
export default {
  name : "Symbols",
  methods : {
    searchButton : function() {},
    ascend : function() {
      this.sortDirection = "ascend";
    },
    descend : function() {
      this.sortDirection = "descend";
    },
    removeSort : function() {
      this.sortDirection = undefined;
    },
    exclude : function(event) {
      this.excludeTickers += " " + event.target.value;
    },
    resetTickers : function() {
      localStorage.excludeTickers = "";
      this.excludeTickers = "";
    }
  },
  data() {
    return {
      companies : [],
      loading : true,
      searchText : "",
      excludeTickers : "",
      sortKey : "symbol",
      sortDirection : undefined,
      searchKey : ["symbol", "companyName"],
      fieldsToFilter : []
    };
  },
  computed : {
    filteredCompanies() {
      const { searchText, searchKey, fieldsToFilter, excludeTickers, sortDirection, sortKey, companies } = this;
      localStorage.excludeTickers = excludeTickers;

      // if base case reached: no searching or filtering applied
      const baseCase = ({ excludeTickers, fieldsToFilter, sortDirection, searchText }) =>
        excludeTickers === ""
          && fieldsToFilter.length === 0
          && sortDirection === undefined
          && searchText === ""

      // sort/filter operations:
      const handleFilter = ({ excludeTickers, fieldsToFilter, sortDirection, sortKey, searchText, searchKey }) => {
        let accum = companies;

        if (excludeTickers.length > 0) {
          accum = reject(accum, excludeTickers, 'symbol');
        }

        if (fieldsToFilter.length > 0) {
          accum = filterCompanies(accum, fieldsToFilter);
        }

        if (sortDirection) {
          accum = sort(accum, sortKey, sortDirection);
        }

        if (searchText !== "") {
          accum = search(accum, searchKey, searchText);
        }

        return accum;
      }

      return baseCase({ excludeTickers, fieldsToFilter, sortDirection, searchText })
        ? this.companies
        : handleFilter({ excludeTickers, fieldsToFilter, sortDirection, sortKey, searchText, searchKey })
    }
  },

  beforeMount() {
    API.getComputerHardwareCompanies()
      .then(response => {
        this.companies = response.data;
        return response.data;
      })
      .finally(() => {
        this.loading = false;
      });
    if (localStorage.excludeTickers) {
      this.excludeTickers = localStorage.excludeTickers;
    }
  }
};
</script>

<style lang="scss" scoped>
@import "../assets/css/_theme";
.company {
  margin-bottom: 10px;
  padding-bottom: 10px;
  border-bottom: 1px solid $white-ter;
}
</style>
