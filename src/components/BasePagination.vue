<template>
  <ul class="catalog__pagination pagination">
    <li class="pagination__item">
      <a class="pagination__link pagination__link--arrow"
         :class="{'pagination__link--disabled': page === 1}"
         href="#" aria-label="Предыдущая страница"
         @click.prevent="paginate(page - 1)">
        <svg width="8" height="14" fill="currentColor">
          <use xlink:href="#icon-arrow-left"></use>
        </svg>
      </a>
    </li>

    <li class="pagination__item" v-for="pageNumber in croppedPagesList" :key="pageNumber">
      <a href="#" class="pagination__link"
         :class="{
          'pagination__link--current': pageNumber === page,
          'pagination__link--display-none': pageNumber === '',
          'pagination__link--default': pageNumber === '...',
          }"
         @click.prevent="paginate(pageNumber)">
        {{ pageNumber }}
      </a>
    </li>

    <li class="pagination__item">
      <a class="pagination__link pagination__link--arrow"
         :class="{'pagination__link--disabled': page >= pages}"
         href="#" aria-label="Следующая страница"
         @click.prevent="paginate(page + 1)">
        <svg width="8" height="14" fill="currentColor">
          <use xlink:href="#icon-arrow-right"></use>
        </svg>
      </a>
    </li>
  </ul>
</template>

<script>

const MAX_PAGES = 5;

export default {
  model: {
    prop: 'page',
    event: 'paginate',
  },
  props: ['page', 'count', 'perPage'],
  computed: {
    maxPages() {
      return MAX_PAGES;
    },
    pages() {
      return Math.ceil(this.count / this.perPage);
    },
    croppedPagesList() {
      if (this.maxPages < this.pages) {
        return [
          this.page > 1 ? 1 : '',
          this.page <= this.pages - 2 ? this.page : '...',
          this.page <= this.pages - 2 ? this.page + 1 : this.page - 1,
          this.page <= this.pages - 2 ? '...' : this.page,
          this.page >= this.pages ? '' : this.pages,
        ];
      }
      return this.pages;
    },
  },
  methods: {
    paginate(page) {
      if (page && page <= this.pages) {
        this.$emit('paginate', page);
      }
    },
  },
};
</script>
