<template>
  <UCard>
    <div class="search-bar">
      <UInput v-model="search" placeholder="Пошук продуктів..." />
    </div>

    <UTable
        :columns="columns"
        :rows="paginatedRows"
        v-model:sort="sort"
        sort-mode="manual"
        class="w-full"
        :ui="{ td: 'max-w-[0] truncate' }"
    >
      <template #thumbnail-data="{ row }">
        <img class="w-[100px] h-[100px]" :src="row.thumbnail" alt="Фото продукту" />
      </template>

      <template #rating-data="{ row }">
        <span :class="row.rating < 4.5 ? 'text-red-700' : 'text-green-700'">
          {{ row.rating }}
        </span>
      </template>
    </UTable>

    <UPagination v-model="page" :total="total" :page-count="pageCount" />
  </UCard>
</template>


<script setup>
import {ref, computed} from 'vue';

useHead({
  title: 'Список продуктів'
});

const columns = [
  {key: 'price', label: 'Ціна', sortable: true},
  {key: 'title', label: 'Назва'},
  {key: 'description', label: 'Опис'},
  {key: 'rating', label: 'Оцінка', sortable: true},
  {key: 'brand', label: 'Бренд'},
  {key: 'category', label: 'Категорія'},
  {key: 'thumbnail', label: 'Фото'}
];

const products = ref([]);
const total = ref(0);
const page = ref(1);
const pageCount = 10;
const search = ref('');
const sort = ref({column: '', direction: ''});

const fetchData = async () => {
  try {
    const response = await fetch('https://dummyjson.com/products');
    const data = await response.json();
    products.value = data.products || [];

    total.value = data.total || 0;
  } catch (error) {
    console.error('Error fetching data:', error);
  }
};

fetchData();

const filteredRows = computed(() => {
  if (!search.value) return products.value;
  page.value = 1;                                     // щоб ми могли бачити дані
  return products.value.filter(product =>
      Object.values(product).some(value =>
          String(value).toLowerCase().includes(search.value.toLowerCase())
      )
  );
});

const sortRows = (a, b) => {
  if (!sort.value.column) return 0;
  const key = sort.value.column;
  const direction = sort.value.direction === 'asc' ? 1 : -1;
  if (a[key] < b[key]) return -1 * direction;
  if (a[key] > b[key]) return 1 * direction;
  return 0;
};

const sortedRows = computed(() => {             //сортуємо за всіма значеннями
  return [...filteredRows.value].sort(sortRows);
});

const paginatedRows = computed(() => {
  const start = (page.value - 1) * pageCount;
  const end = page.value * pageCount;
  return sortedRows.value.slice(start, end);
});
</script>
