<template>
  <div class="container">
    <p>Общее количество товаров : {{totalProducts}} </p>
    <p align="left">Поиск по названию товара: <input type="text" v-model="name" placeholder="Введите название товара" /></p>
    <div>
      <div id="filterBrand">
        <p>Фильтрация по бренду</p>
        <multiselect v-model="checkedBrands" :options="brands" :multiple="true" :close-on-select="false"
                      :clear-on-select="false" :preserve-search="true" placeholder="Выберите необходимые бренды">
        </multiselect>
      </div>
      <div id="filterPriceParent">
        <div id="filterPrice" align="center">
          <p>Фильтрация по цене</p>
          <input class="inputPrice" type="number" v-model="checkedPrices[0]" min="0" :max="checkedPrices[1]" /> -
          <input class="inputPrice" type="number" v-model="checkedPrices[1]" :min="checkedPrices[0]" :max="maxPrice" />
          <vue-slider v-model="checkedPrices" :enable-cross="false" :min="0" :max="maxPrice" tooltipPlacement="bottom"></vue-slider>
        </div>
      </div>
      <button class="btn btn-danger" @click="clearFilters">Сбросить фильтры</button>
    </div>
    <hr>
    <table class="table table-striped">
      <thead class="thead-dark">
        <tr>
          <th scope="col">Артикул</th>
          <th scope="col">Название товара</th>
          <th scope="col">Бренд</th>
          <th scope="col">Вес</th>
          <th scope="col">Фасовка</th>
          <th scope="col">Цена</th>
          <th scope="col">Наличие</th>
        </tr>
      </thead>
      <tbody>
        <ProductItem v-for="item in collection" v-bind:item="item" :key="item.name"></ProductItem>
      </tbody>
    </table>
    <hr>
    <div class="btn-toolbar">
      <div class="btn-group">
        <button :class="['btn btn-secondary', { 'btn-dark' : isActiveLinks[p] }]" v-for="p in numberOfPages" @click.prevent="setPage(p)" :key="p">
          {{p}}
        </button>
      </div>
    </div>
  </div>
</template>

<script>

import productList from '../../data.json'
import _ from 'lodash'
import ProductItem from './ProductItem.vue'
import VueSlider from 'vue-slider-component'
import 'vue-slider-component/theme/default.css'
import Multiselect from 'vue-multiselect'

export default {
  name: 'productList',
  components : {
    ProductItem,
    VueSlider,
    Multiselect
  },
  data() {
    return {
      productList: [],
      perPage : 5,
      pagination: {},
      name : '',
      brands: [],
      checkedBrands: [],
      checkedPrices : [0,0],
      isActiveLinks : [],
      maxPrice: 0,
      value : ''
    }
  },
  computed: {
    filteredList() {
      return this.productList
                  .filter(this.filterByName)
                  .filter(this.filterByBrand)
                  .filter(this.filterByPrice);
    },
    maxPage() {
      return Math.ceil(this.totalProducts / this.perPage);
    },
    collection() {
      var ceil = this.maxPage;
      if ((ceil < this.pagination.currentPage) && (ceil != 0)) this.setPage(ceil);
      return this.paginate(this.filteredList);
    },
    totalProducts() {
      return this.filteredList.length;
    },
    numberOfPages() {
      return _.range(1, this.maxPage + 1)
    }
  },
  methods: {
    filterByPrice(elem) {
      if ((_.parseInt(elem.price) >= this.checkedPrices[0]) && (_.parseInt(elem.price) <= this.checkedPrices[1])) {
        return true;
      }
    },
    filterByBrand(elem) {
      if (this.checkedBrands.length == 0) return true;
      for (let i = 0; i < this.checkedBrands.length; i++) {
        if (elem.brand === this.checkedBrands[i]) {
          return true;
        }
      }
    },
    filterByName(elem) {
      if(this.name==='') return true;
      else return elem.name.indexOf(this.name) > -1;
    },
    setPage(p) {
      _.fill(this.isActiveLinks, false);
      this.isActiveLinks[p] = true;
      this.pagination = this.paginator(this.totalProducts, p);
    },
    paginate(data) {
      return _.slice(data, this.pagination.startIndex, this.pagination.endIndex + 1)
    },
    paginator(totalItems, currentPage) {
      var startIndex = (currentPage - 1) * this.perPage,
      endIndex = Math.min(startIndex + this.perPage - 1, totalItems - 1);
      return {
        currentPage,
        startIndex,
        endIndex
      };
    },
    clearFilters() {
      this.name = '';
      this.checkedBrands = [];
      this.checkedPrices = [0,this.maxPrice];
      this.setPage(1);
    },
    uniqueItem(param) {
      var obj = {};
      for (let i = 0; i < this.productList.length; i++) {
        var str = this.productList[i][param];
        obj[str] = true;
      }
      this[param + 's'] = Object.keys(obj);
    }
  },
  created() {
    this.productList = productList;
    this.uniqueItem('brand');
    this.maxPrice = _.maxBy(productList, function(o) { return _.parseInt(o.price); }).price;
    this.checkedPrices = [0, this.maxPrice];
    this.setPage(1);
  }
}
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>
<style>
  #filterPrice {
    width: 200px;
    margin : auto;
  }
  #filterPriceParent {
    width: 400px;
  }
  #filterBrand {
    align : left;
    width: 400px;
    margin-bottom: 20px;
  }
  .inputPrice {
    width: 90px;
  }
  .multiselect__tag {
    color : #404040;
    background: #a6a6a6;
  }
  .multiselect__tag-icon:focus,
  .multiselect__tag-icon:hover {
    background: #404040;
  }
  .multiselect__tag-icon:after {
    color: #404040;
  }
  .multiselect__option--selected {
    background: #a6a6a6;
    color: #404040;
  }
  .multiselect__option--selected:after {
    color: #eeeeee;
  }
  .multiselect__option--highlight {
    background: #a0c18e;
    color: #eeeeee;
  }
  .multiselect__option--selected.multiselect__option--highlight {
    background: #ca8886;
    color: #eeeeee;
  }
  .multiselect__option--highlight:after {
    width: 170px;
  }
  .vue-slider {
    margin-top: 10px;
  }
  .vue-slider-process {
    background-color : #404040;
  }
  .vue-slider-rail {
    background-color : #a6a6a6;
  }
  .vue-slider-dot-handle {
    border: 2px solid #404040;
  }
  .vue-slider-dot-tooltip-inner {
    color: #404040;
    border-color: #a6a6a6;
    background-color: #a6a6a6;
  }
</style>
