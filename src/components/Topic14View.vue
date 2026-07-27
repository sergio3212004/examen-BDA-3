<script setup lang="ts">
import { computed, ref } from 'vue'

const pipelineStep = ref(0)
const paradigm = ref<'reglas' | 'estadistico' | 'deep'>('reglas')
const lifecycleStep = ref(0)

const pipeline = [
  ['Entrada', '“El banco central aumentó las tasas ayer”', 'Lenguaje natural todavía no estructurado.'],
  ['Análisis léxico', '[El] [banco] [central] [aumentó] [las] [tasas] [ayer]', 'Tokeniza y normaliza. Puede lematizar “aumentó” como “aumentar”.'],
  ['Análisis sintáctico', 'sujeto(banco central) → verbo(aumentó) → objeto(tasas)', 'Representa dependencias y alcance gramatical.'],
  ['Análisis semántico', 'banco = INSTITUCIÓN · tiempo = AYER', 'El contexto desambigua “banco”: no es un asiento.'],
  ['Salida', '{evento: SUBIR_TASA, agente: BANCO_CENTRAL}', 'La interpretación alimenta una clasificación, respuesta, resumen o traducción.'],
]
const paradigms = {
  reglas: { label: 'Basado en reglas', engine: 'SI contiene “banco central” Y “tasas” → FINANZAS', result: 'FINANZAS (regla coincidente)', strength: 'Trazable, determinista y eficaz en dominios acotados.', limit: 'Frágil ante paráfrasis, excepciones y cambios.' },
  estadistico: { label: 'Estadístico', engine: 'P(FINANZAS | banco, tasas, aumentó) = 0.91', result: 'FINANZAS (máxima probabilidad)', strength: 'Aprende de corpus y cuantifica incertidumbre.', limit: 'Depende de atributos y datos representativos.' },
  deep: { label: 'Deep learning', engine: 'tokens → embeddings → Transformer → clasificador', result: 'FINANZAS + evento SUBIR_TASA', strength: 'Modela contexto, matices y transferencia.', limit: 'Costo, opacidad, sesgo y alucinaciones.' },
}
const currentParadigm = computed(() => paradigms[paradigm.value])
const applications = [
  ['Automatización', 'Chatbots atienden consultas rutinarias y derivan casos complejos a personas.'],
  ['Documentos', 'Clasifica, extrae fechas e importes y resume contenido con validación.'],
  ['Traducción', 'Convierte idiomas intentando conservar contexto, tono y matices.'],
  ['Minería de texto', 'Descubre patrones, tendencias y sentimiento en texto no estructurado.'],
  ['Búsqueda semántica', 'Interpreta intención y significado, no solo palabras literales.'],
  ['Generación', 'Los LLM producen contenido condicionado por instrucciones, tono y contexto.'],
]
</script>

<template>
  <section class="page topic14-page">
    <header class="topic14-hero">
      <div><span class="eyebrow">BASE DE DATOS AVANZADA · TEMA 14</span><h1>Procesamiento de <em>Lenguaje Natural.</em></h1><p>El PLN combina lingüística, aprendizaje automático e ingeniería para convertir texto o habla en representaciones que una máquina pueda analizar y transformar en acciones útiles.</p></div>
      <aside><span class="eyebrow">IDEA PARA EL EXAMEN</span><strong>No basta reconocer palabras: el reto es conservar estructura, significado, intención y contexto.</strong><a class="outline-button" href="/[14-1]%20BDA%20-%20Clase.pdf" target="_blank">Abrir PDF original ↗</a></aside>
    </header>

    <section class="t14-definition"><div><span>PLN / NLP</span><h2>¿Qué hace realmente?</h2></div><p>Automatiza traducción, resumen, clasificación, corrección, búsqueda, extracción y generación. “Comprender” significa construir una representación útil para una tarea; no demuestra necesariamente comprensión humana consciente.</p></section>

    <section class="t14-section">
      <span class="eyebrow">LAS CUATRO ETAPAS DEL DOCUMENTO</span><h2>Del símbolo a una salida accionable.</h2>
      <div class="t14-stage-grid"><article v-for="(stage, index) in pipeline.slice(1)" :key="stage[0]"><span>0{{ index + 1 }}</span><h3>{{ stage[0] }}</h3><p>{{ stage[2] }}</p><i v-if="index < 3">→</i></article></div>
      <div class="t14-exam-note"><strong>Distinción clave:</strong> sintaxis pregunta cómo se relacionan las palabras; semántica pregunta qué significa esa estructura. La pragmática añade intención, situación y conocimiento compartido.</div>
    </section>

    <section class="t14-section t14-pipeline-section">
      <div class="section-heading"><div><span class="eyebrow">DIAGRAMA ANIMADO</span><h2>Observa cómo cambia la representación.</h2></div><span class="status">{{ pipelineStep + 1 }} / {{ pipeline.length }}</span></div>
      <div class="t14-pipeline"><button v-for="(step, index) in pipeline" :key="step[0]" :class="{ active: index <= pipelineStep, current: index === pipelineStep }" @click="pipelineStep = index"><span>0{{ index + 1 }}</span><strong>{{ step[0] }}</strong><i v-if="index < pipeline.length - 1">→</i></button></div>
      <Transition name="t14-shift" mode="out-in"><div :key="pipelineStep" class="t14-output"><code>{{ pipeline[pipelineStep][1] }}</code><p>{{ pipeline[pipelineStep][2] }}</p></div></Transition>
      <button class="primary t14-next" @click="pipelineStep = (pipelineStep + 1) % pipeline.length">{{ pipelineStep === pipeline.length - 1 ? 'Reiniciar transformación ↻' : 'Siguiente transformación →' }}</button>
    </section>

    <section class="t14-section">
      <span class="eyebrow">BONDADES Y APLICACIONES</span><h2>El valor aparece al convertir texto no estructurado en decisiones.</h2>
      <div class="t14-app-grid"><article v-for="(item, index) in applications" :key="item[0]"><span>0{{ index + 1 }}</span><h3>{{ item[0] }}</h3><p>{{ item[1] }}</p></article></div>
      <div class="t14-warning"><strong>Sentimiento no es lectura literal:</strong> negación, sarcasmo, ironía, emoción mixta y cultura pueden invertir una clasificación.</div>
    </section>

    <section class="t14-section">
      <span class="eyebrow">EVOLUCIÓN DE ENFOQUES</span><h2>Reglas, estadística y deep learning no son sinónimos.</h2>
      <div class="t14-table"><div class="head"><span>Criterio</span><span>Reglas</span><span>Estadístico</span><span>Deep learning</span></div><div><strong>Conocimiento</strong><span>Escrito por expertos</span><span>Estimado de datos</span><span>Representaciones aprendidas</span></div><div><strong>Ejemplos</strong><span>Patrones, gramáticas</span><span>n-gramas, HMM, regresión</span><span>RNN, Transformer, LLM</span></div><div><strong>Ventaja</strong><span>Explicable y controlable</span><span>Probabilístico y eficiente</span><span>Contexto y transferencia</span></div><div><strong>Límite</strong><span>Cobertura manual</span><span>Ingeniería de atributos</span><span>Costo, opacidad y sesgo</span></div></div>
      <p class="t14-synthesis"><strong>Respuesta de alta calidad:</strong> el enfoque adecuado depende del dominio. Un híbrido puede usar un modelo profundo para interpretar y reglas para imponer restricciones legales o de negocio.</p>
    </section>

    <section class="t14-section t14-lab">
      <span class="eyebrow">LABORATORIO: UNA ENTRADA, TRES PARADIGMAS</span><h2>“El banco central aumentó las tasas ayer”.</h2>
      <div class="t14-tabs"><button v-for="key in (['reglas','estadistico','deep'] as const)" :key="key" :class="{ active: paradigm === key }" @click="paradigm = key">{{ paradigms[key].label }}</button></div>
      <Transition name="t14-shift" mode="out-in"><article :key="paradigm" class="t14-paradigm"><div><span>MOTOR</span><code>{{ currentParadigm.engine }}</code></div><b>→</b><div><span>RESULTADO</span><strong>{{ currentParadigm.result }}</strong></div><aside><p><b>Fortaleza:</b> {{ currentParadigm.strength }}</p><p><b>Límite:</b> {{ currentParadigm.limit }}</p></aside></article></Transition>
    </section>

    <section class="t14-section">
      <span class="eyebrow">CÓMO FUNCIONA LA PNL, SEGÚN EL DOCUMENTO</span><h2>Separa inferencia de entrenamiento.</h2>
      <div class="t14-lifecycle"><button v-for="(name, index) in ['Procesamiento','Extracción','Análisis','Entrenamiento']" :key="name" :class="{ active: index <= lifecycleStep }" @click="lifecycleStep = index"><span>0{{ index + 1 }}</span><strong>{{ name }}</strong><i v-if="index < 3">→</i></button></div>
      <button class="text-button" @click="lifecycleStep = (lifecycleStep + 1) % 4">Avanzar fase →</button>
      <div class="t14-exam-note"><strong>Precisión conceptual:</strong> las tres primeras fases pueden ejecutarse sobre una consulta. El entrenamiento suele ser offline y previo; solo el aprendizaje en línea actualiza continuamente.</div>
    </section>

    <section class="t14-section">
      <span class="eyebrow">DIFICULTAD ALTA</span><h2>Limitaciones, riesgos y postura defendible.</h2>
      <div class="t14-risk-grid"><article><h3>Ambigüedad y cultura</h3><p>Polisemia, referencias, modismos, sarcasmo y contexto permiten interpretaciones distintas.</p></article><article><h3>Sesgo y privacidad</h3><p>El corpus puede reproducir discriminación o contener información sensible.</p></article><article><h3>Alucinación</h3><p>Un LLM optimiza probabilidad del texto, no verdad. Fluidez no equivale a evidencia.</p></article><article><h3>Decisiones críticas</h3><p>En salud o justicia importan errores, explicabilidad, apelación y responsabilidad humana.</p></article></div>
      <blockquote>El PLN amplía el análisis del lenguaje a escala, pero su salida es una inferencia condicionada por datos, objetivos y contexto, no una lectura infalible de la intención humana.</blockquote>
    </section>
  </section>
</template>
