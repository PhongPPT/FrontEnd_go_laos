<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const provinces = ref([])
const districts = ref([])
const villages = ref([])

const loadingProvinces = ref(false)
const loadingDistricts = ref(false)
const loadingVillages = ref(false)

const selectedProvince = ref(null)
const selectedDistrict = ref(null)
const selectedVillage = ref(null)

const errorProvinces = ref(null)
const errorDistricts = ref(null)
const errorVillages = ref(null)

const parseError = (err) => {
  if (err.response && err.response.status === 429) {
    return '⚠️ ທ່ານຮ້ອງຂໍຂໍ້ມູນໄວເກີນໄປ. ກະລຸນາລໍຖ້າຈັກໜ້ອຍແລ້ວລອງໃໝ່.'
  }
  return '🔌 ເກີດຂໍ້ຜິດພາດ ກະລຸນາລອງໃໝ່ພາຍຫລັງ.'
}

const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:8000/api/v1'
const currentEndpoint = ref(`${API_URL}/provinces`)
const copiedId = ref(null)

const copyToClipboard = (text, id) => {
  navigator.clipboard.writeText(text)
  copiedId.value = id
  setTimeout(() => { 
    if (copiedId.value === id) copiedId.value = null 
  }, 2000)
}

const emit = defineEmits(['select-geo'])

const fetchProvinces = async () => {
  loadingProvinces.value = true
  errorProvinces.value = null
  currentEndpoint.value = `${API_URL}/provinces`
  try {
    const res = await axios.get(`${API_URL}/provinces?limit=100`)
    provinces.value = res.data.data.list_data || []
  } catch (err) {
    console.error('Error fetching provinces:', err)
    errorProvinces.value = parseError(err)
  } finally {
    loadingProvinces.value = false
  }
}

const selectProvince = async (prov) => {
  selectedProvince.value = prov
  selectedDistrict.value = null
  selectedVillage.value = null
  districts.value = []
  villages.value = []
  
  if (prov && prov.lat && prov.lng) {
    emit('select-geo', { ...prov, type: 'province' })
  }
  
  loadingDistricts.value = true
  errorDistricts.value = null
  currentEndpoint.value = `${API_URL}/provinces/${prov.id}/districts`
  try {
    const res = await axios.get(`${API_URL}/provinces/${prov.id}/districts?limit=100`)
    districts.value = res.data.data.list_data || []
  } catch (err) {
    console.error('Error fetching districts:', err)
    errorDistricts.value = parseError(err)
  } finally {
    loadingDistricts.value = false
  }
}

const selectDistrict = async (dist) => {
  selectedDistrict.value = dist
  selectedVillage.value = null
  villages.value = []

  if (dist && dist.lat && dist.lng) {
    emit('select-geo', { ...dist, type: 'district' })
  }

  loadingVillages.value = true
  errorVillages.value = null
  currentEndpoint.value = `${API_URL}/districts/${dist.id}/villages`
  try {
    const res = await axios.get(`${API_URL}/districts/${dist.id}/villages?limit=100`)
    villages.value = res.data.data.list_data || []
  } catch (err) {
    console.error('Error fetching villages:', err)
    errorVillages.value = parseError(err)
  } finally {
    loadingVillages.value = false
  }
}

const selectVillage = (vil) => {
  selectedVillage.value = vil
  if (vil && vil.lat && vil.lng) {
    emit('select-geo', { ...vil, type: 'village' })
  }
}

onMounted(() => {
  fetchProvinces()
})
</script>

<template>
  <section class="w-full max-w-6xl px-6 flex flex-col z-10 animate-slide-up" style="animation-delay: 0.2s">
    <div class="flex items-center justify-between mb-4 px-2">
      <h2 class="text-xl font-bold text-slate-800 dark:text-white transition-colors">
        ທົດລອງນຳໃຊ້ (Interactive Demo)
      </h2>
      
      <!-- Current Dynamic Endpoint -->
      <div class="hidden md:flex items-center gap-2 bg-white/60 dark:bg-slate-800/60 backdrop-blur-sm border border-slate-200 dark:border-slate-700 rounded-full px-3 py-1 shadow-sm transition-colors">
        <span class="w-2 h-2 rounded-full animate-pulse transition-colors duration-300"
              :class="(errorProvinces || errorDistricts || errorVillages) ? 'bg-red-500 shadow-[0_0_8px_rgba(239,68,68,0.8)]' : (loadingProvinces || loadingDistricts || loadingVillages) ? 'bg-amber-500 shadow-[0_0_8px_rgba(245,158,11,0.8)]' : 'bg-emerald-500'"></span>
        <code class="text-xs text-slate-500 dark:text-slate-400 font-mono" :class="(errorProvinces || errorDistricts || errorVillages) ? 'text-red-500 dark:text-red-400' : ''">{{ currentEndpoint }}</code>
        <button 
          @click="copyToClipboard(currentEndpoint, 'dynamic')"
          class="text-slate-400 dark:text-slate-500 hover:text-primary dark:hover:text-primary transition-colors ml-2"
          title="Copy Current URL"
        >
          <svg v-if="copiedId !== 'dynamic'" xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect width="14" height="14" x="8" y="8" rx="2" ry="2"/><path d="M4 16c-1.1 0-2-.9-2-2V4c0-1.1.9-2 2-2h10c1.1 0 2 .9 2 2"/></svg>
          <svg v-else class="text-emerald-500 dark:text-emerald-400" xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 6 9 17l-5-5"/></svg>
        </button>
      </div>
    </div>

    <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
      <!-- Provinces Column -->
      <div class="bg-white/70 dark:bg-slate-800/70 backdrop-blur-xl border border-white/40 dark:border-slate-700/50 shadow-xl dark:shadow-2xl rounded-2xl overflow-hidden flex flex-col h-[600px] transition-all duration-300 hover:shadow-2xl hover:-translate-y-1">
        <div class="bg-slate-100/50 dark:bg-slate-900/50 p-4 border-b border-slate-200 dark:border-slate-700 transition-colors">
          <h3 class="text-lg font-bold text-slate-800 dark:text-slate-100 flex items-center gap-2">
            📍 ແຂວງ (Provinces)
          </h3>
        </div>
        <div class="flex-1 overflow-y-auto p-4 space-y-2 custom-scrollbar">
          <div v-if="errorProvinces" class="h-full flex flex-col items-center justify-center text-center px-4 animate-fade-in">
            <span class="text-3xl mb-3 drop-shadow-md">📡</span>
            <div class="flex items-center gap-2 mb-2">
              <span class="w-2.5 h-2.5 rounded-full bg-red-500 animate-pulse shadow-[0_0_8px_rgba(239,68,68,0.8)]"></span>
              <p class="text-red-500 dark:text-red-400 font-bold text-lg">ເກີດຂໍ້ຜິດພາດ!</p>
            </div>
            <p class="text-red-500/80 dark:text-red-400/80 font-medium text-sm mb-4">{{ errorProvinces }}</p>
            <button @click="fetchProvinces" class="px-4 py-2 bg-slate-200 dark:bg-slate-700 hover:bg-slate-300 dark:hover:bg-slate-600 rounded-xl text-sm font-semibold transition-colors">
              🔄 ລອງໃໝ່ອີກຄັ້ງ
            </button>
          </div>
          <div v-else-if="loadingProvinces" class="animate-pulse space-y-3">
            <div class="h-12 bg-slate-200 dark:bg-slate-700 rounded-lg w-full" v-for="i in 5" :key="i"></div>
          </div>
          <div v-else>
            <button 
              v-for="p in provinces" 
              :key="p.id"
              @click="selectProvince(p)"
              :class="['w-full text-left p-4 rounded-xl transition-all duration-200 border', 
                selectedProvince?.id === p.id 
                  ? 'bg-primary text-white border-primary shadow-md transform scale-[1.02]' 
                  : 'bg-white dark:bg-slate-800 hover:bg-slate-50 dark:hover:bg-slate-700 border-slate-100 dark:border-slate-700 hover:border-primary/30 dark:hover:border-primary/50 text-slate-700 dark:text-slate-300']"
            >
              <div class="font-semibold">{{ p.name }}</div>
              <div class="text-sm opacity-80">{{ p.name_en }}</div>
            </button>
          </div>
        </div>
      </div>

      <!-- Districts Column -->
      <div class="bg-white/70 dark:bg-slate-800/70 backdrop-blur-xl border border-white/40 dark:border-slate-700/50 shadow-xl dark:shadow-2xl rounded-2xl overflow-hidden flex flex-col h-[600px] transition-all duration-300 hover:shadow-2xl hover:-translate-y-1" :class="{'opacity-50 pointer-events-none': !selectedProvince}">
        <div class="bg-slate-100/50 dark:bg-slate-900/50 p-4 border-b border-slate-200 dark:border-slate-700 flex justify-between items-center transition-colors">
          <h3 class="text-lg font-bold text-slate-800 dark:text-slate-100 flex items-center gap-2">
            🗺️ ເມືອງ (Districts)
          </h3>
          <span v-if="selectedProvince" class="text-xs font-semibold bg-primary/10 dark:bg-primary/20 text-primary dark:text-blue-400 px-2 py-1 rounded-full">
            ພົບ {{ districts.length }} ລາຍການ
          </span>
        </div>
        <div class="flex-1 overflow-y-auto p-4 space-y-2 custom-scrollbar">
          <div v-if="!selectedProvince" class="h-full flex items-center justify-center text-slate-400 dark:text-slate-500 text-sm">
            ກະລຸນາເລືອກແຂວງກ່ອນ
          </div>
          <div v-else-if="errorDistricts" class="h-full flex flex-col items-center justify-center text-center px-4 animate-fade-in">
            <div class="flex items-center gap-2 mb-2">
              <span class="w-2 h-2 rounded-full bg-red-500 animate-pulse shadow-[0_0_8px_rgba(239,68,68,0.8)]"></span>
              <p class="text-red-500 dark:text-red-400 font-bold">ດຶງຂໍ້ມູນບໍ່ສຳເລັດ</p>
            </div>
            <p class="text-red-500/80 dark:text-red-400/80 font-medium text-sm mb-4">{{ errorDistricts }}</p>
            <button @click="selectProvince(selectedProvince)" class="px-4 py-2 bg-slate-200 dark:bg-slate-700 hover:bg-slate-300 dark:hover:bg-slate-600 rounded-xl text-sm font-semibold transition-colors">
              🔄 ລອງໃໝ່ອີກຄັ້ງ
            </button>
          </div>
          <div v-else-if="loadingDistricts" class="animate-pulse space-y-3">
            <div class="h-12 bg-slate-200 dark:bg-slate-700 rounded-lg w-full" v-for="i in 5" :key="i"></div>
          </div>
          <div v-else-if="districts.length === 0" class="text-center text-slate-500 dark:text-slate-400 mt-10">ບໍ່ພົບຂໍ້ມູນເມືອງ.</div>
          <div v-else>
            <button 
              v-for="d in districts" 
              :key="d.id"
              @click="selectDistrict(d)"
              :class="['w-full text-left p-4 rounded-xl transition-all duration-200 border', 
                selectedDistrict?.id === d.id 
                  ? 'bg-emerald-500 text-white border-emerald-500 shadow-md transform scale-[1.02]' 
                  : 'bg-white dark:bg-slate-800 hover:bg-slate-50 dark:hover:bg-slate-700 border-slate-100 dark:border-slate-700 hover:border-emerald-500/30 dark:hover:border-emerald-500/50 text-slate-700 dark:text-slate-300']"
            >
              <div class="font-semibold">{{ d.name }}</div>
              <div class="text-sm opacity-80">{{ d.name_en }}</div>
            </button>
          </div>
        </div>
      </div>

      <!-- Villages Column -->
      <div class="bg-white/70 dark:bg-slate-800/70 backdrop-blur-xl border border-white/40 dark:border-slate-700/50 shadow-xl dark:shadow-2xl rounded-2xl overflow-hidden flex flex-col h-[600px] transition-all duration-300 hover:shadow-2xl hover:-translate-y-1" :class="{'opacity-50 pointer-events-none': !selectedDistrict}">
        <div class="bg-slate-100/50 dark:bg-slate-900/50 p-4 border-b border-slate-200 dark:border-slate-700 flex justify-between items-center transition-colors">
          <h3 class="text-lg font-bold text-slate-800 dark:text-slate-100 flex items-center gap-2">
            🏘️ ບ້ານ (Villages)
          </h3>
          <span v-if="selectedDistrict" class="text-xs font-semibold bg-emerald-500/10 dark:bg-emerald-500/20 text-emerald-600 dark:text-emerald-400 px-2 py-1 rounded-full">
            ພົບ {{ villages.length }} ລາຍການ
          </span>
        </div>
        <div class="flex-1 overflow-y-auto p-4 space-y-2 custom-scrollbar">
          <div v-if="!selectedDistrict" class="h-full flex items-center justify-center text-slate-400 dark:text-slate-500 text-sm text-center px-4">
            ກະລຸນາເລືອກເມືອງກ່ອນ ເພື່ອເບິ່ງຂໍ້ມູນບ້ານ
          </div>
          <div v-else-if="errorVillages" class="h-full flex flex-col items-center justify-center text-center px-4 animate-fade-in">
            <div class="flex items-center gap-2 mb-2">
              <span class="w-2 h-2 rounded-full bg-red-500 animate-pulse shadow-[0_0_8px_rgba(239,68,68,0.8)]"></span>
              <p class="text-red-500 dark:text-red-400 font-bold">ດຶງຂໍ້ມູນບໍ່ສຳເລັດ</p>
            </div>
            <p class="text-red-500/80 dark:text-red-400/80 font-medium text-sm mb-4">{{ errorVillages }}</p>
            <button @click="selectDistrict(selectedDistrict)" class="px-4 py-2 bg-slate-200 dark:bg-slate-700 hover:bg-slate-300 dark:hover:bg-slate-600 rounded-xl text-sm font-semibold transition-colors">
              🔄 ລອງໃໝ່ອີກຄັ້ງ
            </button>
          </div>
          <div v-else-if="loadingVillages" class="animate-pulse space-y-3">
            <div class="h-12 bg-slate-200 dark:bg-slate-700 rounded-lg w-full" v-for="i in 5" :key="i"></div>
          </div>
          <div v-else-if="villages.length === 0" class="text-center text-slate-500 dark:text-slate-400 mt-10">ບໍ່ພົບຂໍ້ມູນບ້ານ.</div>
          <div v-else>
            <button 
              v-for="v in villages" 
              :key="v.id"
              @click="selectVillage(v)"
              :class="['w-full text-left p-4 rounded-xl transition-all duration-200 border', 
                selectedVillage?.id === v.id 
                  ? 'bg-blue-500 text-white border-blue-500 shadow-md transform scale-[1.02]' 
                  : 'bg-white dark:bg-slate-800 hover:bg-slate-50 dark:hover:bg-slate-700 border-slate-100 dark:border-slate-700 hover:border-blue-500/30 dark:hover:border-blue-500/50 text-slate-700 dark:text-slate-300']"
            >
              <div class="font-semibold">{{ v.name }}</div>
              <div class="text-sm opacity-80">{{ v.name_en }}</div>
            </button>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>
