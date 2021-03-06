<template>
  <main class="content container">
    <div class="content__top content__top--catalog">
      <h1 class="content__title">Каталог</h1>
      <span class="content__info">
        {{ this.countProducts }}
        {{ this.countProducts | declText(['товар', 'товара', 'товаров']) }}
      </span>
    </div>

    <div class="content__catalog">
      <ProductFilter
        v-bind.sync="filters"
      />

      <section class="catalog">
        <ProductList
          :products="products"
          v-if="!productsLoading &&
          !productsLoadingFailed"
        />

        <Preloader v-if="productsLoading" />

        <div class="error-wrapper" v-if="productsLoadingFailed">
          <h2 class="error-heading">
            Что-то пошло не так
          </h2>
          <button class="error-button" @click.prevent="loadProducts">
            перезагрузить
          </button>
        </div>

        <div class="error-wrapper"
          v-if="this.countProducts === 0 && !productsLoading"
        >
          <h2 class="error-heading">
            Пусто
          </h2>
          <button class="error-button"
            @click.prevent="formReset"
          >
            Сбросить Фильтр
          </button>
        </div>

        <BasePagination
          v-model="page"
          :count="countProducts"
          :per-page="productPerPage"
        />
      </section>
    </div>
  </main>
</template>

<script>
import ProductList from '@/components/ProductList.vue';
import BasePagination from '@/components/BasePagination.vue';
import ProductFilter from '@/components/ProductFilter.vue';
import axios from 'axios';
import { API_BASE_URL } from '@/config';
import Preloader from '@/components/Preloader.vue';
import declTextMixin from '@/mixins/declTextMixin';
import Qs from 'qs';

export default {
  components: {
    ProductList,
    BasePagination,
    ProductFilter,
    Preloader,
  },
  data() {
    return {
      filters: {
        filterPriceFrom: 1,
        filterPriceTo: 1000000,
        filterCheckedColor: null,
        filterCategoryId: 0,
        productProps: [],
      },

      page: 1,
      productPerPage: 3,

      productsData: null,

      productsLoading: false,
      productsLoadingFailed: false,
    };
  },
  computed: {
    products() {
      return this.productsData
        ? this.productsData.items.map((product) => ({
          ...product,
          image: product.preview.file.url,
        }))
        : [];
    },
    countProducts() {
      return this.productsData ? this.productsData.pagination.total : [];
    },
  },
  methods: {
    loadProducts() {
      this.productsLoading = true;
      this.productsLoadingFailed = false;
      clearTimeout(this.loadProductTimer);
      this.loadProductTimer = setTimeout(() => {
        axios.get(`${API_BASE_URL}/api/products`, {
          params: {
            categoryId: this.filters.filterCategoryId,
            colorId: this.filters.filterCheckedColor,
            minPrice: this.filters.filterPriceFrom,
            maxPrice: this.filters.filterPriceTo,
            limit: this.productPerPage,
            page: this.page,
            props: this.filters.productProps,
            // props: {
            //   battery_power: ['Встроенный аккумулятор'],
            // },
          },
          /* eslint-disable */
          paramsSerializer: function (params) {
            return Qs.stringify(params, { arrayFormat: 'brackets' });
          },
        })
          .then((response) => {
            this.productsData = response.data;
          })
          .catch(() => {
            this.productsLoadingFailed = true;
          })
          .then(() => {
            this.productsLoading = false;
          });
      });
    },

    formReset() {
      this.filters.filterPriceFrom = 1,
      this.filters.filterPriceTo = 1000000,
      this.filters.filterCheckedColor = null,
      this.filters.filterCategoryId = 0,
      this.filters.productProps = []
    },
  },
  watch: {
    page() {
      this.loadProducts();
    },
    filters: {
      handler() {
        this.loadProducts();
      },
      deep: true,
    },
  },
  mixins: [declTextMixin],
  created() {
    this.loadProducts();
  },
};
</script>
