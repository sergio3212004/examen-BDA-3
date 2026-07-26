<script setup lang="ts">
interface ActivityPhase { phase: string; items: string[][] }
defineProps<{ activities: ActivityPhase[] }>()
</script>

<template>
  <section class="page">
    <header class="page-header activities-header"><div><span class="eyebrow">TEMA 11 · ACTIVIDAD DE APRENDIZAJE · 15 RESPUESTAS</span><h1>Actividades del Tema 11.</h1><p>Respuestas modelo sobre escalabilidad, consistencia, bloqueos, tolerancia a fallos y pruebas distribuidas.</p></div><a href="/[11-1]%20BDA%20-%20Clase.pdf" target="_blank" class="outline-button">Abrir PDF Tema 11 ↗</a></header>
    <nav class="phase-nav">
      <a v-for="(phase, i) in activities" :key="phase.phase" :href="'#phase-' + i"><span>0{{ i + 1 }}</span>{{ phase.phase.split('·')[1] }}</a>
    </nav>
    <section v-for="(phase, pi) in activities" :id="'phase-' + pi" :key="phase.phase" class="activity-phase">
      <div class="phase-heading"><span>0{{ pi + 1 }}</span><div><span class="eyebrow">{{ phase.phase.split('·')[0] }}</span><h2>{{ phase.phase.split('·')[1] }}</h2></div></div>
      <details v-for="(item, i) in phase.items" :key="item[0]" class="activity-item">
        <summary><span>{{ String(i + 1).padStart(2, '0') }}</span><strong>{{ item[0] }}</strong><b>+</b></summary>
        <div><span class="eyebrow">RESPUESTA MODELO</span><p>{{ item[1] }}</p><div class="answer-tip"><strong>Para elevar la respuesta:</strong> agrega un ejemplo propio y explicita qué garantía se sacrifica o qué riesgo se controla.</div></div>
      </details>
    </section>
  </section>
</template>
