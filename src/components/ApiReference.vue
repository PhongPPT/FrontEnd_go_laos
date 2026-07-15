<script setup>
import { ref } from 'vue'

const API_URL = import.meta.env.VITE_API_URL
const copiedId = ref(null)

const copyToClipboard = (text, id) => {
  navigator.clipboard.writeText(text)
  copiedId.value = id
  setTimeout(() => { 
    if (copiedId.value === id) copiedId.value = null 
  }, 2000)
}
</script>

<template>
  <section class="w-full max-w-4xl px-6 mb-12 z-10 animate-slide-up" style="animation-delay: 0.1s">
    <div class="bg-white/70 dark:bg-slate-800/70 backdrop-blur-xl border border-white/40 dark:border-slate-700/50 shadow-xl dark:shadow-2xl rounded-2xl p-6 transition-all hover:shadow-2xl">
      <h2 class="text-xl font-bold text-slate-800 dark:text-white mb-4 flex items-center gap-2 transition-colors">
        📖 API Paths (API Reference)
      </h2>
      <div class="space-y-3">
        
        <!-- API 1: Provinces -->
        <div class="flex flex-col sm:flex-row sm:items-center justify-between bg-white dark:bg-slate-900/50 border border-slate-100 dark:border-slate-700 rounded-xl p-3 hover:border-primary/30 dark:hover:border-primary/50 transition-colors gap-3">
          <div class="flex items-center gap-3">
            <span class="text-xs font-bold bg-primary/10 dark:bg-primary/20 text-primary dark:text-blue-400 px-2 py-1 rounded">GET</span>
            <code class="text-sm text-slate-700 dark:text-slate-300 font-mono break-all">/api/v1/provinces</code>
          </div>
          <div class="flex items-center justify-between sm:justify-end gap-4 w-full sm:w-auto">
            <span class="text-xs text-slate-500 dark:text-slate-400">ດຶງຂໍ້ມູນແຂວງທັງໝົດ</span>
            <button 
              @click="copyToClipboard(`${API_URL}/provinces`, 'api1')"
              class="flex items-center gap-1.5 px-3 py-1.5 rounded-lg text-xs font-medium transition-colors shrink-0 border"
              :class="copiedId === 'api1' ? 'bg-emerald-100 dark:bg-emerald-900/50 text-emerald-700 dark:text-emerald-400 border-transparent' : 'bg-slate-100 dark:bg-slate-800 hover:bg-slate-200 dark:hover:bg-slate-700 text-slate-600 dark:text-slate-300 border-slate-200 dark:border-slate-600'"
            >
              <svg v-if="copiedId !== 'api1'" xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect width="14" height="14" x="8" y="8" rx="2" ry="2"/><path d="M4 16c-1.1 0-2-.9-2-2V4c0-1.1.9-2 2-2h10c1.1 0 2 .9 2 2"/></svg>
              <svg v-else xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 6 9 17l-5-5"/></svg>
              {{ copiedId === 'api1' ? 'Copied' : 'Copy' }}
            </button>
          </div>
        </div>

        <!-- API 2: Districts -->
        <div class="flex flex-col sm:flex-row sm:items-center justify-between bg-white dark:bg-slate-900/50 border border-slate-100 dark:border-slate-700 rounded-xl p-3 hover:border-emerald-500/30 dark:hover:border-emerald-500/50 transition-colors gap-3">
          <div class="flex items-center gap-3">
            <span class="text-xs font-bold bg-primary/10 dark:bg-primary/20 text-primary dark:text-blue-400 px-2 py-1 rounded">GET</span>
            <code class="text-sm text-slate-700 dark:text-slate-300 font-mono break-all">/api/v1/provinces/<span class="text-emerald-600 dark:text-emerald-400">{id}</span>/districts</code>
          </div>
          <div class="flex items-center justify-between sm:justify-end gap-4 w-full sm:w-auto">
            <span class="text-xs text-slate-500 dark:text-slate-400">ດຶງຂໍ້ມູນເມືອງໃນແຂວງ</span>
            <button 
              @click="copyToClipboard(`${API_URL}/provinces/{id}/districts`, 'api2')"
              class="flex items-center gap-1.5 px-3 py-1.5 rounded-lg text-xs font-medium transition-colors shrink-0 border"
              :class="copiedId === 'api2' ? 'bg-emerald-100 dark:bg-emerald-900/50 text-emerald-700 dark:text-emerald-400 border-transparent' : 'bg-slate-100 dark:bg-slate-800 hover:bg-slate-200 dark:hover:bg-slate-700 text-slate-600 dark:text-slate-300 border-slate-200 dark:border-slate-600'"
            >
              <svg v-if="copiedId !== 'api2'" xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect width="14" height="14" x="8" y="8" rx="2" ry="2"/><path d="M4 16c-1.1 0-2-.9-2-2V4c0-1.1.9-2 2-2h10c1.1 0 2 .9 2 2"/></svg>
              <svg v-else xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 6 9 17l-5-5"/></svg>
              {{ copiedId === 'api2' ? 'Copied' : 'Copy' }}
            </button>
          </div>
        </div>

        <!-- API 3: Villages -->
        <div class="flex flex-col sm:flex-row sm:items-center justify-between bg-white dark:bg-slate-900/50 border border-slate-100 dark:border-slate-700 rounded-xl p-3 hover:border-orange-500/30 dark:hover:border-orange-500/50 transition-colors gap-3">
          <div class="flex items-center gap-3">
            <span class="text-xs font-bold bg-primary/10 dark:bg-primary/20 text-primary dark:text-blue-400 px-2 py-1 rounded">GET</span>
            <code class="text-sm text-slate-700 dark:text-slate-300 font-mono break-all">/api/v1/districts/<span class="text-emerald-600 dark:text-emerald-400">{id}</span>/villages</code>
          </div>
          <div class="flex items-center justify-between sm:justify-end gap-4 w-full sm:w-auto">
            <span class="text-xs text-slate-500 dark:text-slate-400">ດຶງຂໍ້ມູນບ້ານໃນເມືອງ</span>
            <button 
              @click="copyToClipboard(`${API_URL}/districts/{id}/villages`, 'api3')"
              class="flex items-center gap-1.5 px-3 py-1.5 rounded-lg text-xs font-medium transition-colors shrink-0 border"
              :class="copiedId === 'api3' ? 'bg-emerald-100 dark:bg-emerald-900/50 text-emerald-700 dark:text-emerald-400 border-transparent' : 'bg-slate-100 dark:bg-slate-800 hover:bg-slate-200 dark:hover:bg-slate-700 text-slate-600 dark:text-slate-300 border-slate-200 dark:border-slate-600'"
            >
              <svg v-if="copiedId !== 'api3'" xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect width="14" height="14" x="8" y="8" rx="2" ry="2"/><path d="M4 16c-1.1 0-2-.9-2-2V4c0-1.1.9-2 2-2h10c1.1 0 2 .9 2 2"/></svg>
              <svg v-else xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 6 9 17l-5-5"/></svg>
              {{ copiedId === 'api3' ? 'Copied' : 'Copy' }}
            </button>
          </div>
        </div>

      </div>
    </div>
  </section>
</template>
