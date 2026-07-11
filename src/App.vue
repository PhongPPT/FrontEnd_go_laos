<script setup>
import ThemeToggle from './components/ThemeToggle.vue'
import AppHeader from './components/AppHeader.vue'
import ApiReference from './components/ApiReference.vue'
import InteractiveDemo from './components/InteractiveDemo.vue'
import AppFooter from './components/AppFooter.vue'
</script>

<template>
  <div class="min-h-screen bg-slate-50 dark:bg-slate-900 flex flex-col items-center py-12 font-sans selection:bg-primary selection:text-white relative overflow-hidden transition-colors duration-500">
    
    <ThemeToggle />
    
    <AppHeader />

    <ApiReference />

    <InteractiveDemo />

    <AppFooter />
    
  </div>
</template>

<style>
/* Base scrollbar styles */
.custom-scrollbar::-webkit-scrollbar {
  width: 6px;
}
.custom-scrollbar::-webkit-scrollbar-track {
  background: transparent;
}
.custom-scrollbar::-webkit-scrollbar-thumb {
  background-color: #cbd5e1;
  border-radius: 20px;
}
.dark .custom-scrollbar::-webkit-scrollbar-thumb {
  background-color: #475569;
}
.custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background-color: #94a3b8;
}
.dark .custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background-color: #64748b;
}

/* Animations */
@keyframes slideUp {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}
.animate-slide-up {
  animation: slideUp 0.6s cubic-bezier(0.16, 1, 0.3, 1) forwards;
  opacity: 0;
}
.animate-fade-in {
  animation: fadeIn 0.8s ease-out forwards;
}
</style>
