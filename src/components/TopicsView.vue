<script setup lang="ts">
import MashupSimulator from './MashupSimulator.vue'

interface Topic {
  index: string; title: string; goal: string; summary: string
  deep: string; example: string; trap: string
}

defineProps<{ topics: Topic[]; activeTopic: number; completed: number[] }>()
defineEmits<{ selectTopic: [index: number]; toggleComplete: [index: number] }>()
</script>

<template>
  <section class="page">
    <header class="page-header"><span class="eyebrow">REPOSITORIO CONCEPTUAL · 06 MÓDULOS</span><h1>Entender antes de memorizar.</h1><p>Cada tema conecta mecanismo, aplicación, caso límite y trampa de examen.</p></header>
    <div class="topic-layout">
      <aside class="topic-index">
        <button v-for="(topic, i) in topics" :key="topic.title" :class="{ active: activeTopic === i }" @click="$emit('selectTopic', i)"><span>{{ topic.index }}</span>{{ topic.title }}</button>
      </aside>
      <article class="reader">
        <div class="reader-heading"><div><span class="eyebrow">MÓDULO {{ topics[activeTopic]!.index }} · DIFICULTAD ALTA</span><h2>{{ topics[activeTopic]!.title }}</h2><p>{{ topics[activeTopic]!.goal }}</p></div><button class="complete" :class="{ done: completed.includes(activeTopic) }" @click="$emit('toggleComplete', activeTopic)">{{ completed.includes(activeTopic) ? '✓ Completado' : 'Marcar completado' }}</button></div>
        <div class="concept-block central"><span>IDEA CENTRAL</span><p>{{ topics[activeTopic]!.summary }}</p></div>
        <div class="reader-columns">
          <div class="concept-block"><span>CÓMO FUNCIONA</span><p>{{ topics[activeTopic]!.deep }}</p></div>
          <div class="concept-block example"><span>EJEMPLO APLICADO</span><p>{{ topics[activeTopic]!.example }}</p></div>
        </div>
        <div class="concept-block trap"><span>⚠ TRAMPA DE EXAMEN</span><p>{{ topics[activeTopic]!.trap }}</p></div>
        <MashupSimulator v-if="activeTopic === 1" />
        <div class="analysis-pattern">
          <span class="eyebrow">PLANTILLA PARA RESPONDER AL PROFESOR</span><h3>Contexto → mecanismo → trade-off → consecuencia</h3>
          <ol><li>Define la condición concreta del escenario.</li><li>Nombra el mecanismo distribuido que interviene.</li><li>Explica qué garantía se gana y cuál se debilita.</li><li>Cierra con una consecuencia observable o una condición límite.</li></ol>
        </div>
        <div class="reader-nav"><button :disabled="activeTopic === 0" @click="$emit('selectTopic', activeTopic - 1)">← Anterior</button><span>{{ activeTopic + 1 }} / {{ topics.length }}</span><button :disabled="activeTopic === topics.length - 1" @click="$emit('selectTopic', activeTopic + 1)">Siguiente →</button></div>
      </article>
    </div>
  </section>
</template>
