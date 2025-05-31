<script setup>
import {ref, onMounted} from 'vue';
import {fetchWrapper} from '@/helpers/fetch-wrapper.js';

const baseUrl = `${import.meta.env.VITE_API_URL}/v1`;

// Данные таблицы и пагинация
const records = ref([]);
const totalCount = ref(0);
const currentPage = ref(1);
const itemsPerPage = ref(10);
const isLoading = ref(false);
const error = ref(null);

// Получение данных с сервера
const fetchRecords = async () => {
  try {
    isLoading.value = true;
    error.value = null;

    const response = await fetchWrapper.get(
      `${baseUrl}/data/list?page=${currentPage.value}&limit=${itemsPerPage.value}`
    );

    records.value = response.records;
    totalCount.value = response.count;
  } catch (err) {
    error.value = err.message || 'Ошибка при загрузке данных';
    console.error('Ошибка:', err);
  } finally {
    isLoading.value = false;
  }
};

// Обработчик изменения страницы
const handlePageChange = (page) => {
  currentPage.value = page;
  fetchRecords();
};

// Обработчик действий
const handleAction = async (itemId, itemType) => {
  console.log(`Действие для элемента с ID: ${itemId}, тип: ${itemType}`);
  // Здесь можно добавить разную логику для разных типов
};

// Загрузка данных при монтировании
onMounted(() => {
  fetchRecords();
});
</script>

<template>
  <main>
    <!-- Состояние загрузки -->
    <div v-if="isLoading" class="text-center py-4">
      Загрузка данных...
    </div>

    <!-- Сообщение об ошибке -->
    <div v-if="error" class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded mb-4">
      {{ error }}
    </div>

    <!-- Таблица с данными -->
    <div v-if="!isLoading && !error" class="relative overflow-x-auto">
      <table class="w-full text-sm text-left rtl:text-right text-gray-500 dark:text-gray-400">
        <thead
          class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400">
        <tr>
          <th scope="col" class="px-6 py-3">#</th>
          <th scope="col" class="px-6 py-3">Тип</th>
          <th scope="col" class="px-6 py-3">Meta</th>
          <th scope="col" class="px-6 py-3">Действия</th>
        </tr>
        </thead>
        <tbody>
        <tr
          v-for="item in records"
          :key="item.id"
          class="bg-white border-b dark:bg-gray-800 dark:border-gray-700 border-gray-200"
        >
          <th scope="row"
              class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap dark:text-white">
            {{ item.id }}
          </th>
          <td class="px-6 py-4">{{ item.type }}</td>
          <td class="px-6 py-4">{{ item.meta?.content }}</td>
          <td class="px-6 py-4">
            <button
              @click="handleAction(item.id, item.type)"
              class="flex items-center font-medium text-blue-600 dark:text-blue-500 hover:underline"
            >
              <template v-if="item.type !== 'binary_data'">
                Просмотр
              </template>
              <template v-else>
                Скачать
              </template>
            </button>
          </td>
        </tr>
        </tbody>
      </table>

      <!-- Пагинация -->
      <div class="flex justify-center mt-4">
        <nav class="inline-flex rounded-md shadow">
          <button
            v-for="page in Math.ceil(totalCount / itemsPerPage)"
            :key="page"
            @click="handlePageChange(page)"
            :class="{
              'px-4 py-2 border border-gray-300 bg-white text-gray-700 hover:bg-gray-50': true,
              'bg-blue-50 border-blue-500 text-blue-600': currentPage === page
            }"
          >
            {{ page }}
          </button>
        </nav>
      </div>

      <!-- Информация о пагинации -->
      <div class="text-sm text-gray-500 dark:text-gray-400 mt-2 text-center">
        Показано {{ records.length }} из {{ totalCount }} записей
      </div>
    </div>
  </main>
</template>
