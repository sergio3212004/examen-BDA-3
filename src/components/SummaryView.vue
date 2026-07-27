<script setup lang="ts">
import ClusterSimulator from './ClusterSimulator.vue'

type Tab = 'resumen' | 'temas' | 'tema12'
type Simulation = 'normal' | 'fallo' | 'particion' | 'recuperado'
interface Topic { index: string; title: string; goal: string; icon: string }

defineProps<{ topics: Topic[]; completed: number[]; simulation: Simulation; logs: string[]; flashQuestion: string }>()
defineEmits<{
  selectTab: [tab: Tab]
  selectTopic: [index: number]
  simulate: [state: Simulation]
}>()
</script>

<template>
  <section class="page">
    <header class="hero">
      <div>
        <span class="eyebrow">LAB SESSION · BASES DE DATOS AVANZADAS</span>
        <h1>Sistemas distribuidos,<br><em>sin memorizar a ciegas.</em></h1>
        <p>Comprende los mecanismos, provoca fallos y practica cómo justificar cada decisión técnica.</p>
      </div>
      <div class="hero-action">
        <div class="objective"><span class="eyebrow">OBJETIVO DE SESIÓN</span><strong>Explicar un failover sin confundir disponibilidad, consenso y consistencia.</strong></div>
        <button class="primary" @click="$emit('selectTopic', 4)">Continuar estudiando <span>→</span></button>
      </div>
    </header>

    <ClusterSimulator :simulation="simulation" :logs="logs" @simulate="$emit('simulate', $event)" />

    <section>
      <div class="section-heading"><div><span class="eyebrow">RUTA DE DOMINIO</span><h2>Seis módulos, una sola arquitectura mental.</h2></div><button class="text-button" @click="$emit('selectTab', 'temas')">Ver todos →</button></div>
      <div class="module-grid">
        <button v-for="(topic, i) in topics" :key="topic.title" class="module-card" @click="$emit('selectTopic', i)">
          <span class="module-number">{{ topic.index }}</span><span class="module-icon">{{ topic.icon }}</span>
          <h3>{{ topic.title }}</h3><p>{{ topic.goal }}</p>
          <div><span>{{ completed.includes(i) ? 'COMPLETADO' : i === 4 ? 'EN CURSO' : 'POR ESTUDIAR' }}</span><b>→</b></div>
        </button>
      </div>
    </section>

    <section class="dashboard-bottom">
      <div class="quick-card">
        <span class="eyebrow">RECUPERACIÓN ACTIVA</span><h2>¿Puedes explicarlo sin mirar?</h2>
        <p>{{ flashQuestion }}</p><button class="primary" @click="$emit('selectTab', 'temas')">Ir al Tema 11 →</button>
      </div>
      <button class="activities-card" @click="$emit('selectTab', 'tema12')">
        <span class="eyebrow">NUEVO MÓDULO</span><strong>Tema 12 · Ecosistema Big Data</strong>
        <p>Contenido, quiz avanzado y actividades resueltas en un solo espacio.</p><span>ABRIR TEMA 12 →</span>
      </button>
    </section>
  </section>
</template>
