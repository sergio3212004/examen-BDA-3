<script setup lang="ts">
const phases = [
  { n:'01', title:'Autoaprendizaje', subtitle:'Investigación individual', q:[
    ['Relación con IA','Relaciona PLN, machine learning y deep learning con ejemplos. Aclara que PLN es el campo y ML/DL son técnicas posibles.'],
    ['LLM e impacto','Define LLM, tokens, Transformer, preentrenamiento e inferencia; analiza aplicaciones y riesgos.'],
    ['Límites humanos','Explica ambigüedad, sarcasmo y contexto cultural con ejemplos propios.'],
    ['Representaciones','Compara Bag of Words, TF‑IDF, embeddings estáticos y contextuales.'],
    ['Ética y sociedad','Construye una matriz riesgo–mitigación para chatbots, sentimiento y generación.']]},
  { n:'02', title:'Discusión', subtitle:'Debate crítico grupal', q:[
    ['¿Comprensión o simulación?','Define “entender” y contrasta competencia funcional, anclaje, intención y conciencia.'],
    ['Sustitución profesional','Distingue tareas de profesiones completas; analiza automatización, complementariedad y responsabilidad.'],
    ['Opiniones en redes','Debate sesgo de muestreo, bots, sarcasmo, idioma, representatividad y consentimiento.'],
    ['Decisiones importantes','Evalúa salud o justicia: costo de error, explicabilidad, revisión, apelación y auditoría.'],
    ['¿Qué enfoque elegir?','Defiende reglas, estadística, deep learning o híbrido según datos, costo, trazabilidad y riesgo.']]},
  { n:'03', title:'Reflexión', subtitle:'Síntesis grupal', q:[
    ['Aprendizajes nuevos','Expongan tres ideas que corrigieron una intuición previa y un ejemplo por idea.'],
    ['Percepción de las máquinas','Diferencien rendimiento, generalización, razonamiento y comprensión consciente.'],
    ['Postura final','Formulen una tesis equilibrada con condiciones concretas de uso responsable.'],
    ['Relevancia profesional','Vinculen PLN con ingestión, índices vectoriales, calidad, linaje, privacidad y búsqueda.'],
    ['Proyecto futuro','Diseñen usuarios, datos, pipeline, métrica, línea base, riesgos y criterio de éxito.']]},
]
</script>
<template>
  <section class="page topic14-activities">
    <header class="activities-header"><div><span class="eyebrow">ACTIVIDAD DE APRENDIZAJE · PDF ORIGINAL</span><h1>Investiga, debate y <em>sintetiza.</em></h1><p>Se conservan las tres fases y las quince preguntas; las guías muestran el nivel esperado.</p></div><a class="outline-button" href="/[14-1]%20BDA%20-%20Clase.pdf" target="_blank">Consultar PDF ↗</a></header>
    <nav class="phase-nav"><a v-for="p in phases" :key="p.n" :href="'#t14-'+p.n"><span>{{ p.n }}</span>{{ p.title }}</a></nav>
    <section v-for="p in phases" :id="'t14-'+p.n" :key="p.n" class="activity-phase"><div class="phase-heading"><span>{{ p.n }}</span><div><span class="eyebrow">{{ p.subtitle }}</span><h2>{{ p.title }}</h2></div></div><details v-for="(item,i) in p.q" :key="item[0]" class="activity-item"><summary><span>{{ String(i+1).padStart(2,'0') }}</span><strong>{{ item[0] }}</strong><b>+</b></summary><div><p>{{ item[1] }}</p><div class="answer-tip">Tesis → concepto técnico → ejemplo → límite o contraargumento → conclusión.</div></div></details></section>
  </section>
</template>
