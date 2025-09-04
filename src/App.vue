<template>
  <div class="min-h-screen bg-neutral-50 dark:bg-neutral-900 text-neutral-900 dark:text-neutral-100 p-4 sm:p-8">
    <header class="text-center mb-8">
      <h1 class="text-3xl sm:text-4xl font-bold text-neutral-600 dark:text-neutral-400">💸 Bantay Presyo 💸</h1>
      <p class="text-base sm:text-lg mt-2 text-neutral-600 dark:text-neutral-400">Monitor prevailing retail prices in the Philippines</p>
    </header>

    <main class="max-w-5xl mx-auto">
      <!-- Date Selection and Search -->
      <div class="bg-white dark:bg-neutral-800 rounded-lg shadow-md p-4 sm:p-6 mb-6">
        <div class="flex flex-col sm:flex-row items-center gap-4">
          <div class="relative w-full sm:w-64">
            <VueDatePicker
              v-model="selectedDate"
              :enable-time-picker="false"
              format="MMMM d, yyyy"
              :disabled="loading"
              placeholder="Select a date"
              class="w-full bg-white dark:bg-neutral-800 border border-neutral-300 dark:border-neutral-700 rounded-md px-4 py-2 focus:outline-none focus:ring-2 focus:ring-neutral-500"
              :options="availableDates.map(link => new Date(link.date))"
            />
          </div>
          <button
            @click="fetchData"
            :disabled="loading || !selectedDate"
            class="w-full sm:w-auto bg-neutral-600 text-white px-4 sm:px-6 py-2 rounded-md hover:bg-neutral-700 disabled:opacity-50 disabled:cursor-not-allowed transition"
          >
            <span v-if="loading" class="flex items-center">
              <svg class="animate-spin h-5 w-5 mr-2" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" />
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z" />
              </svg>
              Loading...
            </span>
            <span v-else>Search</span>
          </button>
          <button
            v-if="error"
            @click="fetchAvailableDates"
            class="w-full sm:w-auto bg-yellow-600 text-white px-4 sm:px-6 py-2 rounded-md hover:bg-yellow-700 transition"
          >
            Retry
          </button>
        </div>
        <div class="mt-4 relative">
          <input
            v-model="searchQuery"
            type="text"
            placeholder="Search commodities..."
            class="w-full bg-white dark:bg-neutral-800 border border-neutral-300 dark:border-neutral-700 rounded-md pl-10 pr-4 py-2 focus:outline-none focus:ring-2 focus:ring-neutral-500"
          />
          <svg class="absolute left-3 top-1/2 transform -translate-y-1/2 h-5 w-5 text-neutral-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
          </svg>
        </div>
      </div>

      <!-- Error Message -->
      <div v-if="error" class="bg-red-100 dark:bg-red-900 border-l-4 border-red-500 dark:border-red-700 text-red-700 dark:text-red-300 p-4 rounded-lg mb-6">
        <p class="font-semibold">Error</p>
        <p>{{ error }}</p>
      </div>

      <!-- Loading State (Skeleton) -->
      <div v-if="loading" class="space-y-4">
        <div class="bg-white dark:bg-neutral-800 rounded-lg shadow-md p-4 animate-pulse">
          <div class="h-6 bg-neutral-200 dark:bg-neutral-700 rounded w-1/4 mb-4"></div>
          <div class="space-y-2">
            <div v-for="n in 5" :key="n" class="flex space-x-4">
              <div class="h-4 bg-neutral-200 dark:bg-neutral-700 rounded w-1/4"></div>
              <div class="h-4 bg-neutral-200 dark:bg-neutral-700 rounded w-1/2"></div>
              <div class="h-4 bg-neutral-200 dark:bg-neutral-700 rounded w-1/4"></div>
            </div>
          </div>
        </div>
      </div>

      <!-- Price Data by Category -->
      <div v-if="priceData.length > 0 && !loading" class="space-y-6">
        <div v-for="(categoryItems, category) in groupedPriceData" :key="category" class="bg-white dark:bg-neutral-800 rounded-lg shadow-md">
          <h2 class="text-lg sm:text-xl font-semibold text-neutral-900 dark:text-neutral-100 bg-neutral-100 dark:bg-neutral-700 px-6 py-3 rounded-t-lg">
            {{ category }}
          </h2>
          <div class="overflow-x-auto">
            <table class="min-w-full">
              <thead class="bg-neutral-50 dark:bg-neutral-700">
                <tr>
                  <th class="px-4 sm:px-6 py-3 text-left text-sm font-semibold text-neutral-900 dark:text-neutral-100">Number</th>
                  <th class="px-4 sm:px-6 py-3 text-left text-sm font-semibold text-neutral-900 dark:text-neutral-100">Commodity</th>
                  <th class="px-4 sm:px-6 py-3 text-left text-sm font-semibold text-neutral-900 dark:text-neutral-100">Specification</th>
                  <th class="px-4 sm:px-6 py-3 text-left text-sm font-semibold text-neutral-900 dark:text-neutral-100">Price (₱/UNIT)</th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="item in categoryItems"
                  :key="item.number"
                  class="border-t border-neutral-200 dark:border-neutral-700 hover:bg-neutral-50 dark:hover:bg-neutral-600"
                >
                  <td class="px-4 sm:px-6 py-4">{{ item.number }}</td>
                  <td class="px-4 sm:px-6 py-4">{{ item.commodity }}</td>
                  <td class="px-4 sm:px-6 py-4">{{ item.specification || 'N/A' }}</td>
                  <td class="px-4 sm:px-6 py-4 font-medium">{{ item.price }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- No Data Message -->
      <p v-else-if="!loading && selectedDate" class="text-center py-8 text-neutral-500 dark:text-neutral-400 bg-white dark:bg-neutral-800 rounded-lg shadow-md">
        No data available for the selected date.
      </p>

      <!-- Footer -->
      <footer class="mt-8 text-sm text-neutral-600 dark:text-neutral-400 bg-white dark:bg-neutral-800 rounded-lg shadow-md p-4 sm:p-6">
        <p>Prevailing price is defined as the average price at which any basic necessity has been sold in a given area, computed using the arithmetic mean formula.</p>
        <p class="mt-4 font-semibold">Covered Markets:</p>
        <ol class="list-decimal list-inside ml-4">
          <li>Commonwealth Market</li>
          <li>New Las Piñas City Public Market</li>
          <li>Marikina Public Market</li>
          <li>Muñoz Market</li>
          <li>Pasay City Public Market</li>
          <li>Mutya ng Pasig Mega Market</li>
          <li>Quinta Market</li>
        </ol>
        <p class="mt-4">
          Data Source: 
          <a href="https://www.da.gov.ph/price-monitoring/" target="_blank" class="text-blue-600 dark:text-blue-400 hover:underline">
            Department of Agriculture Price Monitoring
          </a>
        </p>
      </footer>
    </main>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import axios from 'axios';
import VueDatePicker from 'vue3-datepicker';

const apiUrl = import.meta.env.VITE_API_URL || 'https://bantay-presyo-api.vercel.app';
const availableDates = ref([]);
const selectedDate = ref(null); // Changed to Date object for date picker
const priceData = ref([]);
const loading = ref(false);
const error = ref(null);
const searchQuery = ref('');

const formatDate = (date) => {
  if (!date) return '';
  return date.toLocaleDateString('en-US', { month: 'long', day: 'numeric', year: 'numeric' });
};

// Group price data by category
const groupedPriceData = computed(() => {
  const grouped = {};
  const filtered = searchQuery.value
    ? priceData.value.filter(item =>
        item.commodity.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
        item.specification.toLowerCase().includes(searchQuery.value.toLowerCase())
      )
    : priceData.value;

  filtered.forEach(item => {
    const category = item.category || 'UNKNOWN';
    if (!grouped[category]) grouped[category] = [];
    grouped[category].push(item);
  });

  // Sort categories alphabetically
  return Object.fromEntries(
    Object.entries(grouped).sort(([a], [b]) => a.localeCompare(b))
  );
});

const fetchAvailableDates = async (retries = 3, delay = 1000) => {
  for (let attempt = 1; attempt <= retries; attempt++) {
    try {
      loading.value = true;
      error.value = null;
      const response = await axios.get(`${apiUrl}/daily_links?limit=20`, {
        timeout: 10000,
      });
      availableDates.value = response.data;
      if (availableDates.value.length > 0) {
        selectedDate.value = new Date(availableDates.value[0].date); // Default to latest
        await fetchData(); // Auto-fetch for default
      } else {
        error.value = 'No dates available. Please try again later or check backend logs.';
      }
      return;
    } catch (err) {
      console.error(`Attempt ${attempt} failed:`, err);
      if (attempt === retries) {
        error.value = `Failed to load dates: ${err.message}. Please check your network or contact support.`;
      }
      await new Promise(resolve => setTimeout(resolve, delay));
    } finally {
      loading.value = false;
    }
  }
};

const fetchData = async () => {
  if (!selectedDate.value) return;
  try {
    loading.value = true;
    error.value = null;
    const formattedDate = formatDate(selectedDate.value);
    const response = await axios.get(`${apiUrl}/data?date=${encodeURIComponent(formattedDate)}`, {
      timeout: 10000,
    });
    priceData.value = response.data.data || [];
    if (priceData.value.length === 0) {
      error.value = 'No data found for the selected date.';
    }
  } catch (err) {
    console.error('Data fetch error:', err);
    error.value = `Failed to fetch data: ${err.message}. Please try a different date.`;
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  fetchAvailableDates();
});
</script>