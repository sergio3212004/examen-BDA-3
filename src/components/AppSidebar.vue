<script setup lang="ts">
type Tab = 'resumen' | 'temas' | 'tema12'

defineProps<{
  activeTab: Tab
  progress: number
  completedCount: number
  topicCount: number
}>()

defineEmits<{ select: [tab: Tab] }>()

const navigation: { tab: Tab; icon: string; label: string }[] = [
  { tab: 'resumen', icon: '⌂', label: 'Resumen' },
  { tab: 'temas', icon: '11', label: 'Tema 11' },
  { tab: 'tema12', icon: '12', label: 'Tema 12' },
]
</script>

<template>
  <aside class="sidebar">
    <button class="brand" aria-label="Ir al resumen" @click="$emit('select', 'resumen')">
      <span class="brand-icon">▤</span>
      <span><small>BDA</small><strong>STUDY LAB</strong></span>
    </button>
    <nav class="main-nav" aria-label="Navegación principal">
      <button
        v-for="item in navigation"
        :key="item.tab"
        :class="{ active: activeTab === item.tab }"
        @click="$emit('select', item.tab)"
      >
        <span>{{ item.icon }}</span>{{ item.label }}
      </button>
    </nav>
    <div class="side-progress">
      <div><span class="eyebrow">PROGRESO GENERAL</span><strong>{{ progress }}%</strong></div>
      <div class="progress"><span :style="{ width: progress + '%' }"></span></div>
      <small>{{ completedCount }} de {{ topicCount }} módulos completados</small>
    </div>
  </aside>
</template>
