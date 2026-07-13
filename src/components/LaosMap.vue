<script setup>
import { onMounted, ref, watch } from 'vue'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'

const isLayerMenuOpen = ref(false)

// Fix missing marker icons in Leaflet when using Webpack/Vite
import iconRetinaUrl from 'leaflet/dist/images/marker-icon-2x.png'
import iconUrl from 'leaflet/dist/images/marker-icon.png'
import shadowUrl from 'leaflet/dist/images/marker-shadow.png'

delete L.Icon.Default.prototype._getIconUrl
L.Icon.Default.mergeOptions({
  iconRetinaUrl,
  iconUrl,
  shadowUrl,
})

const mapContainer = ref(null)
const map = ref(null)

// Mock Data for Laos Provinces (we will use this since API doesn't have lat/lng yet)
const mockProvinces = [
  { id: 1, name_en: 'Vientiane Capital', name_la: 'ນະຄອນຫຼວງວຽງຈັນ', lat: 17.9757, lng: 102.6331 },
  { id: 2, name_en: 'Phongsaly', name_la: 'ຜົ້ງສາລີ', lat: 21.6833, lng: 102.1000 },
  { id: 3, name_en: 'Luang Namtha', name_la: 'ຫຼວງນໍ້າທາ', lat: 20.9499, lng: 101.4000 },
  { id: 4, name_en: 'Oudomxay', name_la: 'ອຸດົມໄຊ', lat: 20.7333, lng: 101.9833 },
  { id: 5, name_en: 'Bokeo', name_la: 'ບໍ່ແກ້ວ', lat: 20.2833, lng: 100.4167 },
  { id: 6, name_en: 'Luang Prabang', name_la: 'ຫຼວງພະບາງ', lat: 19.8833, lng: 102.1333 },
  { id: 7, name_en: 'Houaphanh', name_la: 'ຫົວພັນ', lat: 20.4167, lng: 104.0000 },
  { id: 8, name_en: 'Xayaboury', name_la: 'ໄຊຍະບູລີ', lat: 19.2500, lng: 101.7500 },
  { id: 9, name_en: 'Xiengkhouang', name_la: 'ຊຽງຂວາງ', lat: 19.3333, lng: 103.3333 },
  { id: 10, name_en: 'Vientiane Province', name_la: 'ວຽງຈັນ', lat: 18.8333, lng: 102.3333 },
  { id: 11, name_en: 'Borikhamxay', name_la: 'ບໍລິຄຳໄຊ', lat: 18.3833, lng: 103.9167 },
  { id: 12, name_en: 'Khammouane', name_la: 'ຄຳມ່ວນ', lat: 17.5500, lng: 105.2167 },
  { id: 13, name_en: 'Savannakhet', name_la: 'ສະຫວັນນະເຂດ', lat: 16.5500, lng: 104.7500 },
  { id: 14, name_en: 'Salavan', name_la: 'ສາລະວັນ', lat: 15.7167, lng: 106.4167 },
  { id: 15, name_en: 'Sekong', name_la: 'ເຊກອງ', lat: 15.3333, lng: 106.7167 },
  { id: 16, name_en: 'Champasak', name_la: 'ຈຳປາສັກ', lat: 14.8833, lng: 105.8833 },
  { id: 17, name_en: 'Attapeu', name_la: 'ອັດຕະປື', lat: 14.8000, lng: 106.8333 },
  { id: 18, name_en: 'Xaysomboun', name_la: 'ໄຊສົມບູນ', lat: 18.8167, lng: 103.0833 },
]

const props = defineProps({
  selectedGeo: {
    type: Object,
    default: null
  }
})

let activeMarker = null

const currentTile = ref('osm')
const tiles = {
  osm: {
    name: 'Standard',
    url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
    attribution: '© OpenStreetMap contributors',
    icon: '🗺️'
  },
  satellite: {
    name: 'Satellite',
    url: 'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}',
    attribution: 'Tiles &copy; Esri &mdash; Source: Esri, i-cubed, USDA, USGS, AEX, GeoEye, Getmapping, Aerogrid, IGN, IGP, UPR-EGP, and the GIS User Community',
    icon: '🛰️'
  },
  terrain: {
    name: 'Terrain',
    url: 'https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png',
    attribution: 'Map data: © OpenStreetMap contributors, SRTM | Map style: © OpenTopoMap (CC-BY-SA)',
    icon: '⛰️'
  }
}

let activeTileLayer = null

const changeTile = (tileKey) => {
  if (activeTileLayer && map.value) {
    map.value.removeLayer(activeTileLayer)
  }
  currentTile.value = tileKey
  isLayerMenuOpen.value = false
  const tileConfig = tiles[tileKey]
  activeTileLayer = L.tileLayer(tileConfig.url, {
    maxZoom: 19,
    attribution: tileConfig.attribution
  }).addTo(map.value)
}

const createPopupHTML = (title, subtitle, typeStr, lat, lng) => `
  <div class="custom-popup-content flex flex-col items-center p-3 font-sans min-w-[150px]">
    <h3 class="font-bold text-xl text-slate-800 mb-1 text-center w-full">${title}</h3>
    ${subtitle ? `<p class="text-sm text-slate-500 m-0 mb-3 text-center w-full">${subtitle}</p>` : ''}
    <div class="mb-4">
      <span class="inline-flex items-center justify-center px-3 py-1 text-sm font-medium text-blue-600 bg-blue-50 rounded-lg">
        ${typeStr}
      </span>
    </div>
    <div class="text-[11px] text-slate-400 font-mono text-center w-full border-t border-slate-100 pt-3">
      [ ${Number(lat).toFixed(6)}, ${Number(lng).toFixed(6)} ]
    </div>
  </div>
`

watch(() => props.selectedGeo, (newGeo) => {
  if (newGeo && newGeo.lat && newGeo.lng && map.value) {
    let zoomLevel = 8 // Default for province
    let geoTypeStr = 'ແຂວງ'
    
    if (newGeo.type === 'district') {
      zoomLevel = 11
      geoTypeStr = 'ເມືອງ'
    } else if (newGeo.type === 'village') {
      zoomLevel = 14
      geoTypeStr = 'ບ້ານ'
    }
    
    // Zoom map
    map.value.flyTo([newGeo.lat, newGeo.lng], zoomLevel, {
      duration: 1.5,
      easeLinearity: 0.25
    })
    
    // Remove previous active marker if exists
    if (activeMarker) {
      map.value.removeLayer(activeMarker)
    }
    
    // Add new marker
    activeMarker = L.marker([newGeo.lat, newGeo.lng]).addTo(map.value)
    
    // Bind popup and open it
    const title = newGeo.name || newGeo.name_la || 'ທີ່ຕັ້ງ'
    const subtitle = newGeo.name_en || ''
    
    activeMarker.bindPopup(createPopupHTML(title, subtitle, geoTypeStr, newGeo.lat, newGeo.lng), {
      className: 'modern-popup',
      closeButton: true,
      minWidth: 200
    }).openPopup()
  }
}, { deep: true })

onMounted(() => {
  // Initialize map
  map.value = L.map(mapContainer.value, {
    zoomControl: false // We can add custom zoom control if needed, or leave default
  }).setView([18.5, 103.5], 6)

  // Add default zoom control to bottom right
  L.control.zoom({ position: 'bottomright' }).addTo(map.value)

  // Add initial tile layer
  changeTile('osm')

  // Add initial mock markers for provinces
  mockProvinces.forEach(province => {
    const marker = L.marker([province.lat, province.lng]).addTo(map.value)
    marker.bindPopup(createPopupHTML(province.name_la, province.name_en, 'ແຂວງ', province.lat, province.lng), {
      className: 'modern-popup',
      closeButton: true,
      minWidth: 200
    })
  })
})
</script>

<template>
  <section class="w-full max-w-[1200px] mt-8 px-6 animate-slide-up relative z-10" style="animation-delay: 0.2s;">
    <div class="mb-6 flex justify-between items-end">
      <div>
        <h2 class="text-2xl font-bold text-slate-800 dark:text-slate-100 flex items-center gap-2">
          <span class="text-3xl">🗺️</span>
          ແຜນທີ່ແບບໂຕ້ຕອບໄດ້ (Interactive)
        </h2>
        <p class="text-slate-500 dark:text-slate-400 mt-1">ສຳຫຼວດແຂວງຕ່າງໆ ຂອງປະເທດລາວ</p>
      </div>
    </div>

    <!-- Map Card -->
    <div class="bg-white dark:bg-slate-800/80 rounded-2xl shadow-xl shadow-black/5 dark:shadow-black/20 border border-slate-200 dark:border-slate-700/50 p-2 overflow-hidden backdrop-blur-sm transition-colors duration-300 relative">
      
      <!-- Map Type Switcher -->
      <div 
        class="absolute top-6 left-6 z-[400]" 
        @mouseleave="isLayerMenuOpen = false"
      >
        <!-- Toggle Button -->
        <button 
          @click="isLayerMenuOpen = !isLayerMenuOpen"
          @mouseenter="isLayerMenuOpen = true"
          class="bg-white/90 dark:bg-slate-800/90 backdrop-blur-md p-2 rounded-xl shadow-lg border border-slate-200/50 dark:border-slate-700/50 text-slate-700 dark:text-slate-300 hover:text-blue-500 dark:hover:text-blue-400 transition-colors flex items-center justify-center w-10 h-10"
          title="Change Map Style"
        >
          <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polygon points="3 6 9 3 15 6 21 3 21 18 15 21 9 18 3 21"/><line x1="9" y1="3" x2="9" y2="18"/><line x1="15" y1="6" x2="15" y2="21"/></svg>
        </button>

        <!-- Menu Options -->
        <div 
          v-show="isLayerMenuOpen"
          class="absolute top-0 left-12 bg-white/90 dark:bg-slate-800/90 backdrop-blur-md p-1.5 rounded-xl shadow-lg border border-slate-200/50 dark:border-slate-700/50 flex flex-col gap-1 w-max transition-all duration-200 origin-left"
        >
          <button 
            v-for="(config, key) in tiles" 
            :key="key"
            @click="changeTile(key)"
            class="px-3 py-2 rounded-lg text-sm font-medium flex items-center gap-2 transition-all duration-200"
            :class="currentTile === key 
              ? 'bg-blue-500 text-white shadow-md' 
              : 'text-slate-600 dark:text-slate-300 hover:bg-slate-100 dark:hover:bg-slate-700'"
          >
            <span class="text-lg leading-none">{{ config.icon }}</span>
            {{ config.name }}
          </button>
        </div>
      </div>

      <div 
        ref="mapContainer" 
        class="w-full h-[550px] rounded-xl z-0 font-sans"
      ></div>
    </div>
  </section>
</template>

<style scoped>
/* Ensure map controls stay under our theme toggles and tooltips */
:deep(.leaflet-control-container) {
  z-index: 400 !important;
}

:deep(.leaflet-pane) {
  z-index: 200 !important;
}

:deep(.leaflet-top),
:deep(.leaflet-bottom) {
  z-index: 400 !important;
}

/* Modern Popup Styles */
:deep(.modern-popup .leaflet-popup-content-wrapper) {
  background: white;
  border-radius: 16px;
  box-shadow: 0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1);
  padding: 0;
  overflow: hidden;
  border: 1px solid rgba(0,0,0,0.05);
}

:deep(.modern-popup .leaflet-popup-content) {
  margin: 0;
  line-height: 1.5;
}

:deep(.modern-popup .leaflet-popup-tip) {
  background: white;
  box-shadow: 2px 2px 5px rgba(0,0,0,0.05);
}

:deep(.modern-popup .leaflet-popup-close-button) {
  color: #94a3b8;
  padding: 8px 8px 0 0;
  width: 24px;
  height: 24px;
  font-size: 20px;
  transition: color 0.2s;
}

:deep(.modern-popup .leaflet-popup-close-button:hover) {
  color: #334155;
  background: transparent;
}
</style>
