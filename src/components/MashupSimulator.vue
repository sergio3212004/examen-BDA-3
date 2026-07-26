<script setup lang="ts">
import { computed, ref } from 'vue'

const sources = [
  { id: 'trafico', icon: '🚗', name: 'Tráfico', data: 'Velocidad y congestión' },
  { id: 'clima', icon: '🌦', name: 'Clima', data: 'Lluvia y temperatura' },
  { id: 'ubicacion', icon: '📍', name: 'Ubicación', data: 'Posición del usuario' },
  { id: 'eventos', icon: '📅', name: 'Eventos', data: 'Afluencia y horarios' },
]

const selected = ref(['trafico', 'clima', 'ubicacion'])
const stage = ref(0)

const selectedSources = computed(() => sources.filter(source => selected.value.includes(source.id)))
const result = computed(() => {
  if (selected.value.includes('trafico') && selected.value.includes('clima') && selected.value.includes('ubicacion')) {
    return { title: 'Ruta inteligente', text: 'Recomienda una ruta segura considerando la ubicación, la congestión y el clima.' }
  }
  if (selected.value.includes('trafico') && selected.value.includes('eventos')) {
    return { title: 'Predicción de congestión', text: 'Anticipa zonas saturadas combinando movilidad y concentración de personas.' }
  }
  if (selected.value.includes('clima') && selected.value.includes('eventos')) {
    return { title: 'Planificador de actividades', text: 'Sugiere actividades compatibles con el pronóstico y la agenda local.' }
  }
  return { title: 'Servicio básico', text: 'Selecciona más fuentes para producir una recomendación derivada más completa.' }
})

function toggleSource(id: string) {
  selected.value = selected.value.includes(id)
    ? selected.value.filter(item => item !== id)
    : [...selected.value, id]
  stage.value = 0
}

function advance() {
  stage.value = stage.value === 3 ? 0 : stage.value + 1
}
</script>

<template>
  <section class="mashup-learning">
    <div class="mashup-reference">
      <div>
        <span class="eyebrow">FIGURA DE LA DIAPOSITIVA 8</span>
        <h3>De datos dispersos a un servicio nuevo</h3>
        <p>La imagen de la clase muestra que un mash-up no es una fuente de datos aislada. Primero se capturan datos de la web, vehículos, sensores, hospitales y otros sistemas; después una plataforma distribuida los almacena y analiza; finalmente, sus resultados se combinan para ofrecer servicios como navegación, predicción meteorológica o prevención del delito.</p>
      </div>
      <figure>
        <img src="/mashup-diagrama-slide-08.png" alt="Diagrama de la diapositiva 8: adquisición de datos, procesamiento distribuido, análisis y servicios mash-up">
        <figcaption>Fuente: “Clases”, diapositiva 8, material BDA del profesor Juan Orlando Salazar Campos.</figcaption>
      </figure>
    </div>

    <div class="mashup-simulator">
      <div class="section-heading">
        <div><span class="eyebrow">SIMULACIÓN INTERACTIVA</span><h3>Construye un mash-up paso a paso</h3></div>
        <span class="mashup-step">PASO {{ stage + 1 }} / 4</span>
      </div>

      <div class="mashup-pipeline">
        <section :class="{ active: stage === 0, complete: stage > 0 }">
          <span class="pipeline-number">01</span><h4>Fuentes</h4>
          <p>Elige datos independientes.</p>
          <div class="source-options">
            <button v-for="source in sources" :key="source.id" :class="{ selected: selected.includes(source.id) }" @click="toggleSource(source.id)">
              <span>{{ source.icon }}</span><strong>{{ source.name }}</strong><small>{{ source.data }}</small>
            </button>
          </div>
        </section>

        <span class="pipeline-arrow">→</span>
        <section :class="{ active: stage === 1, complete: stage > 1 }">
          <span class="pipeline-number">02</span><h4>Adquisición</h4>
          <p>Las API normalizan los datos.</p>
          <div class="data-stream">
            <span v-for="source in selectedSources" :key="source.id">{{ source.icon }} JSON</span>
            <small v-if="!selectedSources.length">Sin entradas</small>
          </div>
        </section>

        <span class="pipeline-arrow">→</span>
        <section :class="{ active: stage === 2, complete: stage > 2 }">
          <span class="pipeline-number">03</span><h4>Análisis</h4>
          <p>La plataforma cruza las señales.</p>
          <div class="analysis-engine"><span>⚙</span><strong>Filtrar → correlacionar → predecir</strong><small>{{ selectedSources.length }} fuentes procesadas</small></div>
        </section>

        <span class="pipeline-arrow">→</span>
        <section :class="{ active: stage === 3 }">
          <span class="pipeline-number">04</span><h4>Mash-up</h4>
          <p>Una capacidad nueva para el usuario.</p>
          <div class="mashup-result"><span>✨ SERVICIO GENERADO</span><strong>{{ result.title }}</strong><small>{{ result.text }}</small></div>
        </section>
      </div>

      <div class="mashup-controls">
        <p><strong>{{ stage === 0 ? 'Selecciona las fuentes.' : stage === 1 ? 'Los formatos distintos se convierten a una estructura común.' : stage === 2 ? 'El análisis encuentra relaciones que ninguna fuente contiene por sí sola.' : 'El usuario recibe un servicio derivado, no una simple lista de datos.' }}</strong></p>
        <button class="primary" :disabled="!selectedSources.length" @click="advance">{{ stage === 3 ? 'Reiniciar simulación ↻' : 'Ejecutar siguiente paso →' }}</button>
      </div>
    </div>
  </section>
</template>
