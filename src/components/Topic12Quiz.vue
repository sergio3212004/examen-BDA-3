<script setup lang="ts">
import { computed, ref } from 'vue'

const questions = [
  { area: 'Flink · tiempo', q: 'Un banco recibe eventos con retrasos variables. ¿Qué diseño conserva mejor la corrección de ventanas de fraude?', options: ['Kafka y conteo según hora de llegada', 'Flink con tiempo de evento, watermarks y estado checkpointed', 'HDFS con un archivo por evento', 'Calcite como almacén de eventos'], correct: 1, explanation: 'El tiempo de evento separa cuándo ocurrió el hecho de cuándo llegó. Los watermarks expresan cuánto retraso se tolera y los checkpoints recuperan estado y posiciones coherentes.' },
  { area: 'Spark · recuperación', q: 'Spark pierde una partición de un RDD. ¿Por qué puede recuperarla sin replicar cada resultado intermedio?', options: ['El Driver conserva todos los bytes', 'El linaje describe cómo recomputarla desde sus dependencias', 'Kafka convierte el RDD en topic', 'YARN corrige el algoritmo'], correct: 1, explanation: 'El linaje conserva las transformaciones deterministas. Spark reejecuta la rama necesaria, aunque un linaje largo puede justificar checkpoints.' },
  { area: 'Kafka · paralelismo', q: 'Se añaden brokers, pero un topic mantiene una sola partición. ¿Qué conclusión es correcta?', options: ['El grupo escala ilimitadamente', 'Las réplicas son unidades adicionales de consumo', 'El paralelismo del grupo continúa limitado a un consumidor activo', 'Cada consumidor recibe eventos diferentes'], correct: 2, explanation: 'En un grupo, una partición se asigna a un consumidor. Las réplicas aportan disponibilidad, no más unidades lógicas de paralelismo.' },
  { area: 'Drill · esquema', q: '¿Cuándo schema-on-read se convierte en una desventaja?', options: ['Al consultar JSON', 'Cuando variaciones deben detectarse antes de consultas críticas', 'Cuando existe un nodo adicional', 'Cuando el esquema es flexible'], correct: 1, explanation: 'Schema-on-read aplaza validación e interpretación. Para datos críticos puede ser preferible imponer contratos antes de servir consultas.' },
  { area: 'Presto · optimización', q: 'Presto une una tabla pequeña con una enorme fuente remota. ¿Qué decisión puede reducir más el costo?', options: ['Cambiar a NoSQL sin medir', 'Empujar filtros y seleccionar una distribución de join que minimice transferencia', 'Eliminar el Coordinator', 'Replicar cada fila'], correct: 1, explanation: 'El movimiento de datos domina muchas consultas distribuidas. Predicate pushdown y una estrategia de join apropiada evitan transferencias innecesarias.' },
  { area: 'Flink · exactly-once', q: 'Un checkpoint termina, pero el sumidero escribe dos veces después de reiniciar. ¿Qué faltó?', options: ['Más memoria', 'Idempotencia o coordinación transaccional de extremo a extremo', 'Schema-on-read', 'Más particiones HDFS'], correct: 1, explanation: 'Exactly-once dentro del motor no garantiza exactamente un efecto externo. El sink debe participar en la transacción o deduplicar con una clave estable.' },
  { area: 'Kafka + Flink', q: '¿Por qué Kafka y Flink no son sustitutos directos?', options: ['Ambos solo almacenan archivos', 'Kafka conserva/distribuye eventos y Flink computa sobre ellos con estado y tiempo', 'Flink no consume eventos', 'Kafka ejecuta cualquier ML'], correct: 1, explanation: 'Son complementarios: Kafka actúa como log durable y Flink como procesador de streams con estado.' },
  { area: 'Spark · memoria', q: 'Un equipo elige Spark únicamente porque “trabaja en memoria”. ¿Cuál es la mejor objeción?', options: ['La RAM siempre es lenta', 'Deben medirse reutilización, shuffle, spill, latencia y costo operacional', 'Spark no es distribuido', 'Hadoop nunca utiliza RAM'], correct: 1, explanation: 'La memoria ayuda cuando se reutilizan datos, pero shuffle, serialización y derrame pueden dominar. La elección debe partir del perfil real de la carga.' },
  { area: 'Hadoop · localidad', q: '¿Qué cambio debilita más la ventaja clásica de localidad de datos de Hadoop?', options: ['Ejecutar el cómputo donde están los bloques', 'Separar cómputo y almacenamiento mediante una red saturada', 'Replicar bloques', 'Usar archivos grandes'], correct: 1, explanation: 'Si cada tarea debe transferir grandes volúmenes por una red saturada, acercar el cómputo al dato deja de cumplirse y el cuello cambia a la red.' },
  { area: 'Calcite · responsabilidad', q: 'Una empresa instala Calcite y espera que almacene petabytes. ¿Qué error conceptual existe?', options: ['Calcite solo acepta NoSQL', 'Calcite optimiza y planifica consultas, pero no es un motor de almacenamiento', 'Calcite no entiende SQL', 'Calcite reemplaza Kafka'], correct: 1, explanation: 'Calcite aporta parser, álgebra relacional, reglas y optimización basada en costos. Otro sistema ejecuta y almacena.' },
  { area: 'Samza · partición', q: 'Un perfil por usuario produce resultados inconsistentes porque eventos del mismo usuario llegan a tareas distintas. ¿Qué revisar primero?', options: ['La clave de partición', 'La tipografía del dashboard', 'El tamaño del Driver Spark', 'El esquema de HDFS'], correct: 0, explanation: 'El estado por clave exige que los eventos relacionados se enruten consistentemente a la misma partición/tarea, salvo coordinación adicional.' },
  { area: 'Arquitectura', q: 'Una pyme procesa 20 000 ventas diarias y acepta reportes al día siguiente. ¿Qué decisión es más defendible?', options: ['Desplegar Kafka, Flink, Storm y Samza', 'Comenzar con base relacional y job batch medido', 'Usar Flink porque streaming siempre es superior', 'Fragmentar cada venta entre diez brokers'], correct: 1, explanation: 'La arquitectura debe ser proporcional al requisito. Con bajo volumen y latencia diaria, un diseño simple reduce costo y riesgo sin sacrificar la necesidad real.' },
]

const started = ref(false)
const finished = ref(false)
const index = ref(0)
const answers = ref<(number | null)[]>(Array(questions.length).fill(null))
const score = computed(() => questions.reduce((total, question, i) => total + (answers.value[i] === question.correct ? 1 : 0), 0))

function start() {
  answers.value = Array(questions.length).fill(null)
  index.value = 0
  finished.value = false
  started.value = true
  window.scrollTo({ top: 0, behavior: 'smooth' })
}
</script>

<template>
  <section class="page topic12-page">
    <header class="page-header"><span class="eyebrow">QUIZ TEMA 12 · DIFICULTAD ALTA</span><h1>Arquitectura antes que memoria.</h1><p>Doce escenarios para comprobar si sabes elegir y combinar frameworks según sus garantías.</p></header>
    <section class="quiz-section">
      <div v-if="!started" class="quiz-intro">
        <div><span class="eyebrow">EVALUACIÓN DE TRANSFERENCIA</span><h2>Decide, compara y detecta límites.</h2><p>Los distractores contienen afirmaciones plausibles. Identifica la responsabilidad de cada tecnología y la garantía que realmente ofrece.</p><ul><li>12 preguntas de análisis</li><li>Calificación automática</li><li>Feedback completo al finalizar</li></ul></div>
        <button class="primary quiz-start" @click="start">Iniciar Quiz Tema 12 <span>→</span></button>
      </div>
      <div v-else-if="!finished">
        <div class="quiz-top"><div><span class="eyebrow">TEMA 12 · PREGUNTA {{ index + 1 }} DE {{ questions.length }}</span><h2>{{ questions[index]!.area }}</h2></div><strong>{{ Math.round(((index + 1) / questions.length) * 100) }}%</strong></div>
        <div class="quiz-progress"><span :style="{ width: ((index + 1) / questions.length) * 100 + '%' }"></span></div>
        <article class="quiz-card"><span class="difficulty">DIFICULTAD ALTA · ANÁLISIS</span><h3>{{ questions[index]!.q }}</h3><div class="quiz-options"><button v-for="(option, oi) in questions[index]!.options" :key="option" :class="{ selected: answers[index] === oi }" @click="answers[index] = oi"><span>{{ String.fromCharCode(65 + oi) }}</span><strong>{{ option }}</strong></button></div></article>
        <div class="quiz-nav"><button :disabled="index === 0" @click="index--">← Anterior</button><div class="quiz-dots"><button v-for="(_, qi) in questions" :key="qi" :class="{ current: index === qi, answered: answers[qi] !== null }" @click="index = qi"></button></div><button v-if="index < questions.length - 1" :disabled="answers[index] === null" @click="index++">Siguiente →</button><button v-else class="finish" :disabled="answers.some(answer => answer === null)" @click="finished = true">Calificar</button></div>
      </div>
      <div v-else class="quiz-results">
        <header><span class="eyebrow">RESULTADO · QUIZ TEMA 12</span><div class="score-ring"><strong>{{ score }}</strong><span>/ {{ questions.length }}</span></div><h2>{{ score >= 10 ? 'Dominio sólido' : score >= 8 ? 'Buen criterio con puntos por revisar' : score >= 6 ? 'Comprensión inestable' : 'Reconstruye las responsabilidades' }}</h2><p>{{ Math.round(score / questions.length * 100) }}% de precisión. Abre cada pregunta para revisar tu respuesta, la correcta y su fundamento.</p><button class="outline-button" @click="start">Reintentar quiz ↻</button></header>
        <div class="result-list"><details v-for="(question, qi) in questions" :key="question.q" :class="{ correct: answers[qi] === question.correct, wrong: answers[qi] !== question.correct }"><summary><span>{{ answers[qi] === question.correct ? '✓' : '×' }}</span><strong>{{ qi + 1 }}. {{ question.q }}</strong><b>+</b></summary><div><p><span>TU RESPUESTA</span>{{ question.options[answers[qi] ?? 0] }}</p><p><span>RESPUESTA CORRECTA</span>{{ question.options[question.correct] }}</p><p><span>POR QUÉ</span>{{ question.explanation }}</p></div></details></div>
      </div>
    </section>
  </section>
</template>
