<template>
  <div class="min-h-screen bg-[#FDFDFD] text-slate-900 font-sans p-4 md:p-12">
    <div class="max-w-5xl mx-auto">
      <header class="mb-12 text-center md:text-left">
        <h1 class="text-4xl font-black tracking-tight text-slate-800">
          Bantay<span class="text-emerald-600">Presyo</span>
        </h1>
        <p class="text-slate-500 font-medium mt-1">NCR Agri-Fishery Retail Prices</p>
      </header>

      <div class="bg-white border border-slate-200 shadow-sm rounded-3xl p-6 mb-10 flex flex-col md:flex-row gap-4 items-center">
        <div class="w-full md:w-72 border p-2.5 rounded-lg">
          <VueDatePicker v-model="selectedDate" :enable-time-picker="false" placeholder="Select Date" class=""/>
        </div>
        <button @click="fetchData" :disabled="loading" 
          class="w-full md:w-auto px-8 py-2.5 bg-slate-900 text-white rounded-xl font-bold hover:bg-emerald-600 transition-all disabled:opacity-50">
          {{ loading ? 'Updating...' : 'Search' }}
        </button>
        <div class="flex-grow relative w-full">
          <input v-model="searchQuery" placeholder="Search commodity (e.g. Rice, Tilapia)..." 
            class="w-full pl-10 pr-4 py-2.5 bg-slate-50 border border-slate-200 rounded-xl focus:ring-2 focus:ring-emerald-500 outline-none transition" />
          <span class="absolute left-3 top-3 text-slate-400">🔍</span>
        </div>
      </div>

      <div v-if="Object.keys(groupedData).length" class="space-y-12">
        <section v-for="(items, cat) in groupedData" :key="cat" class="animate-in">
          <div class="flex items-center gap-3 mb-6">
            <div class="h-px flex-grow bg-slate-200"></div>
            <h2 class="text-xs font-black tracking-widest text-slate-400 uppercase whitespace-nowrap">{{ cat }}</h2>
            <div class="h-px flex-grow bg-slate-200"></div>
          </div>
          <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-5">
            <div v-for="item in items" :key="item.commodity" 
              class="bg-white border border-slate-100 p-5 rounded-2xl hover:border-emerald-200 hover:shadow-md transition-all group">
              <div class="flex justify-between items-start gap-4">
                <div>
                  <h3 class="font-bold text-slate-700 leading-tight">{{ item.commodity }}</h3>
                  <p class="text-xs text-slate-400 mt-1 uppercase">{{ item.specification || 'Standard' }}</p>
                </div>
                <div class="text-right">
                  <span class="text-lg font-black text-slate-900">₱{{ item.price }}</span>
                </div>
              </div>
            </div>
          </div>
        </section>
      </div>

      <footer class="mt-20 pt-10 border-t border-slate-200 text-center">
        <div class="space-y-4 text-sm text-slate-500">
          <p>
            Data Source: 
            <a href="https://www.da.gov.ph/price-monitoring/" target="_blank" class="text-emerald-600 font-bold hover:underline">
              Department of Agriculture Price Monitoring
            </a>
          </p>
          <p>
            Spotted a problem or want to share feedback? Reach out to me on 
            <a href="https://www.facebook.com/joshsarmiento22/" target="_blank" class="text-emerald-600 font-bold hover:underline">
              Facebook
            </a>.
          </p>
          <div class="pt-4 border-t border-slate-100 max-w-xs mx-auto">
            <p class="italic text-xs opacity-75">
              This is an independent project and not affiliated with the Department of Agriculture.
            </p>
            <p class="font-bold mt-2 text-slate-400">
              © 2025 BantayPresyo. All rights reserved.
            </p>
          </div>
        </div>
      </footer>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import axios from 'axios';
import VueDatePicker from 'vue3-datepicker';

const apiUrl = 'http://localhost:3000';
const apiKey = 'vldqKFnIG2IHawV8lPsOjEgoG6zmkEay7u7f2IUr5pGQL9bO63PkU0iCVZPwRQ4atO1sX86Yt2LYqwjFjQKD8Ek835apFjgjWGY4mrkhA0CB0Xbwm1YOWi86KKbLc5nK';

const selectedDate = ref(new Date());
const priceData = ref([]);
const loading = ref(false);
const searchQuery = ref('');

const groupedData = computed(() => {
  const grouped = {};
  const query = searchQuery.value.toLowerCase();
  
  priceData.value.filter(i => i.commodity.toLowerCase().includes(query)).forEach(item => {
    if (!grouped[item.category]) grouped[item.category] = [];
    grouped[item.category].push(item);
  });
  
  // Alphabetical sort within categories
  Object.keys(grouped).forEach(k => grouped[k].sort((a,b) => a.commodity.localeCompare(b.commodity)));
  return grouped;
});

const fetchData = async () => {
  loading.value = true;
  try {
    const formatted = selectedDate.value.toLocaleDateString('en-US', { month: 'long', day: 'numeric', year: 'numeric' });
    const { data } = await axios.get(`${apiUrl}/proxy`, {
      params: { endpoint: 'data', date: formatted },
      headers: { Authorization: `Bearer ${apiKey}` }
    });
    priceData.value = data.data;
  } catch (e) {
    alert("Error fetching data. Ensure the server is running.");
  } finally {
    loading.value = false;
  }
};

onMounted(fetchData);
</script>

<style>
/* Smooth transitions for the UI */
.animate-in { animation: fadeIn 0.5s ease-out; }
@keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
</style>