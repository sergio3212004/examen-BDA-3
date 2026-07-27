<script setup lang="ts">
import { computed, ref } from 'vue'

type Lab = 'processing' | 'kafka' | 'recovery' | 'sql'
type Engine = 'hadoop' | 'spark' | 'flink'

const lab = ref<Lab>('processing')
const processingMode = ref<'batch' | 'stream'>('batch')
const processingStep = ref(0)
const kafkaEvents = ref<{ id: number; key: string; partition: number }[]>([])
const kafkaSequence = ref(1)
const engine = ref<Engine>('spark')
const recoveryStep = ref(0)
const sqlStep = ref(0)

const processing = {
  batch: {
    title: 'Procesamiento batch',
    latency: 'Minutos u horas',
    input: 'Conjunto limitado',
    state: 'Resultados entre etapas',
    steps: ['Acumular 24 h de ventas', 'Leer bloques distribuidos', 'Procesar particiones en paralelo', 'Combinar y publicar reporte'],
    example: 'Una aseguradora recalcula cada noche el riesgo de toda su cartera. Importa terminar 30 TB de forma eficiente, no responder cada milisegundo.',
    trap: 'Batch no significa “obsoleto”: es adecuado cuando la entrada está acotada y el throughput domina la latencia.',
  },
  stream: {
    title: 'Procesamiento streaming',
    latency: 'Milisegundos o segundos',
    input: 'Flujo potencialmente ilimitado',
    state: 'Continuo, por clave y ventana',
    steps: ['Recibir pago', 'Asignar tiempo de evento', 'Actualizar ventana y estado', 'Emitir alerta o resultado'],
    example: 'Uber utiliza Kafka y Flink en pipelines que agregan eventos de anuncios y coordinan checkpoints con escrituras transaccionales.',
    trap: 'Streaming no implica que cada evento llegue ordenado ni que exactamente-una-vez aparezca automáticamente.',
  },
}

const recovery = {
  hadoop: {
    name: 'Hadoop MapReduce',
    unit: 'Bloque y tarea',
    mechanism: 'Réplica HDFS + reejecución',
    stages: ['DataNode deja de responder', 'NameNode identifica bloques afectados', 'Otra réplica sigue disponible', 'La tarea se reejecuta cerca de una copia'],
    insight: 'Se recuperan datos mediante réplica y trabajo mediante reejecución. Es robusto, pero repetir etapas puede aumentar latencia.',
  },
  spark: {
    name: 'Apache Spark',
    unit: 'Partición del DAG',
    mechanism: 'Linaje + checkpoint opcional',
    stages: ['Executor pierde una partición', 'El Driver consulta el linaje', 'Reejecuta dependencias necesarias', 'Reconstruye solo la partición perdida'],
    insight: 'El linaje evita replicar cada resultado intermedio. Un linaje largo o una fuente no reproducible puede requerir checkpoint.',
  },
  flink: {
    name: 'Apache Flink',
    unit: 'Estado + offsets',
    mechanism: 'Checkpoint consistente',
    stages: ['Falla un operador con estado', 'Se detiene y redistribuye el job', 'Restaura snapshot y posiciones', 'Reproduce eventos posteriores al checkpoint'],
    insight: 'La recuperación debe mantener coherencia entre estado y entrada. Exactly-once real también exige un sink transaccional o idempotente.',
  },
}

const currentProcessing = computed(() => processing[processingMode.value])
const currentRecovery = computed(() => recovery[engine.value])

function nextProcessing() {
  processingStep.value = processingStep.value === 4 ? 0 : processingStep.value + 1
}

function addKafkaEvent(key: string) {
  const partition = [...key].reduce((sum, char) => sum + char.charCodeAt(0), 0) % 3
  kafkaEvents.value = [...kafkaEvents.value.slice(-8), { id: kafkaSequence.value++, key, partition }]
}

function nextRecovery() {
  recoveryStep.value = recoveryStep.value === 4 ? 0 : recoveryStep.value + 1
}

function nextSql() {
  sqlStep.value = sqlStep.value === 4 ? 0 : sqlStep.value + 1
}
</script>

<template>
  <section class="t12-deep-lab">
    <div class="section-heading"><div><span class="eyebrow">LABORATORIO CONCEPTUAL · NIVEL EXAMEN</span><h2>Comprende lo que sucede dentro del clúster.</h2></div></div>
    <nav class="deep-lab-tabs">
      <button :class="{ active: lab === 'processing' }" @click="lab = 'processing'">Batch vs. streaming</button>
      <button :class="{ active: lab === 'kafka' }" @click="lab = 'kafka'">Kafka y particiones</button>
      <button :class="{ active: lab === 'recovery' }" @click="lab = 'recovery'">Recuperación</button>
      <button :class="{ active: lab === 'sql' }" @click="lab = 'sql'">SQL distribuido</button>
    </nav>

    <Transition name="deep-lab" mode="out-in">
      <article v-if="lab === 'processing'" key="processing" class="deep-lab-panel">
        <div class="lab-heading">
          <div><span class="eyebrow">MODELO DE EJECUCIÓN</span><h3>{{ currentProcessing.title }}</h3></div>
          <div class="mode-toggle"><button :class="{ active: processingMode === 'batch' }" @click="processingMode = 'batch'; processingStep = 0">Batch</button><button :class="{ active: processingMode === 'stream' }" @click="processingMode = 'stream'; processingStep = 0">Streaming</button></div>
        </div>
        <div class="processing-metrics"><div><span>ENTRADA</span><strong>{{ currentProcessing.input }}</strong></div><div><span>LATENCIA OBJETIVO</span><strong>{{ currentProcessing.latency }}</strong></div><div><span>ESTADO</span><strong>{{ currentProcessing.state }}</strong></div></div>
        <div class="processing-animation" :class="processingMode">
          <div v-for="(item, index) in currentProcessing.steps" :key="item" :class="{ active: processingStep >= index + 1 }"><span>0{{ index + 1 }}</span><strong>{{ item }}</strong><i v-if="index < 3">→</i></div>
        </div>
        <div class="lab-controls"><div><p><strong>Ejemplo:</strong> {{ currentProcessing.example }}</p><p class="exam-trap"><strong>Trampa:</strong> {{ currentProcessing.trap }}</p></div><button class="primary" @click="nextProcessing">{{ processingStep === 4 ? 'Reiniciar ↻' : 'Ejecutar etapa →' }}</button></div>
      </article>

      <article v-else-if="lab === 'kafka'" key="kafka" class="deep-lab-panel">
        <div class="lab-heading"><div><span class="eyebrow">LOG DISTRIBUIDO</span><h3>La clave decide orden y paralelismo.</h3><p>Publica eventos. La misma clave siempre llega a la misma partición, conservando orden local.</p></div></div>
        <div class="kafka-actions"><button @click="addKafkaEvent('cliente-A')">+ Compra cliente A</button><button @click="addKafkaEvent('cliente-B')">+ Compra cliente B</button><button @click="addKafkaEvent('cliente-C')">+ Compra cliente C</button><button @click="kafkaEvents = []">Limpiar</button></div>
        <div class="kafka-cluster">
          <div v-for="partition in 3" :key="partition" class="kafka-partition"><span>PARTICIÓN {{ partition - 1 }}</span><TransitionGroup name="kafka-event"><div v-for="event in kafkaEvents.filter(item => item.partition === partition - 1)" :key="event.id"><b>#{{ event.id }}</b>{{ event.key }}</div></TransitionGroup></div>
          <div class="consumer-group"><span>GRUPO DE CONSUMIDORES</span><div>C1 ← P0</div><div>C2 ← P1</div><div>C3 ← P2</div></div>
        </div>
        <div class="concept-grid"><div><span>ORDEN</span><p>Kafka garantiza orden dentro de una partición, no orden global entre todas.</p></div><div><span>PARALELISMO</span><p>Un grupo no puede usar más consumidores activos que particiones.</p></div><div><span>RÉPLICA</span><p>Las réplicas mejoran disponibilidad; no crean nuevas unidades de consumo.</p></div></div>
        <div class="real-system"><strong>Caso LinkedIn:</strong> Kafka nació para desacoplar productores y múltiples consumidores de actividad, métricas y logs a gran escala. La decisión de partición afecta directamente orden, balance y capacidad.</div>
      </article>

      <article v-else-if="lab === 'recovery'" key="recovery" class="deep-lab-panel">
        <div class="lab-heading"><div><span class="eyebrow">TOLERANCIA A FALLOS COMPARADA</span><h3>{{ currentRecovery.name }}</h3></div><div class="mode-toggle"><button v-for="value in (['hadoop','spark','flink'] as Engine[])" :key="value" :class="{ active: engine === value }" @click="engine = value; recoveryStep = 0">{{ value }}</button></div></div>
        <div class="recovery-summary"><div><span>UNIDAD RECUPERADA</span><strong>{{ currentRecovery.unit }}</strong></div><div><span>MECANISMO</span><strong>{{ currentRecovery.mechanism }}</strong></div></div>
        <div class="recovery-animation"><div v-for="(stage, index) in currentRecovery.stages" :key="stage" :class="{ active: recoveryStep >= index + 1, failed: index === 0 && recoveryStep > 0 }"><span>{{ index + 1 }}</span><strong>{{ stage }}</strong><i v-if="index < 3">→</i></div></div>
        <div class="lab-controls"><p><strong>Lectura de examen:</strong> {{ currentRecovery.insight }}</p><button class="primary" @click="nextRecovery">{{ recoveryStep === 4 ? 'Reiniciar ↻' : 'Inyectar siguiente evento →' }}</button></div>
      </article>

      <article v-else key="sql" class="deep-lab-panel">
        <div class="lab-heading"><div><span class="eyebrow">DRILL · PRESTO · CALCITE</span><h3>Una consulta SQL no es un único trabajo.</h3><p>Observa cómo una consulta se convierte en un plan distribuido.</p></div></div>
        <div class="sql-query">SELECT ciudad, SUM(total) FROM lake.ventas JOIN mysql.clientes USING (cliente_id) WHERE fecha = CURRENT_DATE GROUP BY ciudad;</div>
        <div class="sql-plan">
          <div :class="{ active: sqlStep >= 1 }"><span>01</span><strong>Parsear y validar</strong><small>Calcite puede convertir SQL en álgebra relacional.</small></div>
          <div :class="{ active: sqlStep >= 2 }"><span>02</span><strong>Optimizar</strong><small>Empuja filtros y elige orden/distribución de joins.</small></div>
          <div :class="{ active: sqlStep >= 3 }"><span>03</span><strong>Fragmentar</strong><small>Coordinator crea etapas para múltiples workers.</small></div>
          <div :class="{ active: sqlStep >= 4 }"><span>04</span><strong>Ejecutar y reunir</strong><small>Workers leen fuentes y agregan resultados parciales.</small></div>
        </div>
        <div class="lab-controls"><div><p><strong>Presto:</strong> consulta federada interactiva mediante conectores.</p><p><strong>Drill:</strong> exploración schema-on-read de formatos como JSON y Parquet.</p><p class="exam-trap"><strong>Trampa:</strong> Calcite optimiza planes, pero no almacena petabytes ni sustituye al motor ejecutor.</p></div><button class="primary" @click="nextSql">{{ sqlStep === 4 ? 'Reiniciar ↻' : 'Construir plan →' }}</button></div>
      </article>
    </Transition>
  </section>
</template>
