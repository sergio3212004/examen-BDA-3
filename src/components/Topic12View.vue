<script setup lang="ts">
import { computed, ref } from 'vue'

type Category = 'todos' | 'batch' | 'stream' | 'mensajeria' | 'sql' | 'ml'

const category = ref<Category>('todos')
const activeTool = ref('Spark')
const scenario = ref<'fraude' | 'reportes' | 'exploracion'>('fraude')
const pipelineStep = ref(0)

const tools = [
  { name: 'Hadoop', category: 'batch', badge: 'BATCH + STORAGE', icon: '◫', summary: 'Procesamiento por lotes y almacenamiento distribuido sobre hardware común.', architecture: 'HDFS divide y replica bloques; MapReduce paraleliza el trabajo; YARN asigna recursos; Common aporta utilidades compartidas.', strength: 'Throughput, localidad de datos y recuperación mediante réplica o reejecución.', limit: 'Alta latencia para interacción, iteración y eventos que exigen respuesta inmediata.', example: 'Procesar durante la noche años de transacciones para recalcular indicadores regulatorios.' },
  { name: 'Spark', category: 'batch', badge: 'MEMORIA + DAG', icon: '✦', summary: 'Motor unificado para batch, SQL, streaming y machine learning con reutilización en memoria.', architecture: 'El Driver construye un DAG y coordina executors mediante YARN o Kubernetes. Los RDD conservan linaje para reconstruir particiones perdidas.', strength: 'Algoritmos iterativos y pipelines que reutilizan datos intermedios.', limit: 'Mantener datos en memoria no elimina el costo de shuffle, serialización ni derrame a disco.', example: 'Entrenar iterativamente un modelo de recomendación sobre interacciones de millones de usuarios.' },
  { name: 'Flink', category: 'stream', badge: 'STREAM + ESTADO', icon: '≈', summary: 'Procesa flujos ilimitados con estado, baja latencia y semántica de tiempo de evento.', architecture: 'Un grafo fuente–transformación–sumidero mantiene estado distribuido. Checkpoints consistentes permiten recuperación; watermarks razonan sobre datos tardíos.', strength: 'Ventanas, eventos fuera de orden y aplicaciones continuas con estado.', limit: 'La corrección depende de configurar estado, tiempo, checkpoints y sumideros de manera coherente.', example: 'Detectar fraude mientras llegan pagos móviles desordenados desde distintas regiones.' },
  { name: 'Kafka', category: 'mensajeria', badge: 'EVENT LOG', icon: '⇄', summary: 'Registro distribuido de eventos que desacopla productores y consumidores.', architecture: 'Los productores publican en topics particionados y replicados entre brokers. Los consumidores avanzan mediante offsets dentro de grupos.', strength: 'Durabilidad, replay, integración y escalado del flujo por particiones.', limit: 'Kafka transporta y conserva eventos; por sí solo no reemplaza un motor analítico con estado.', example: 'Centralizar compras, clics e inventario para alimentar varios servicios sin acoplarlos.' },
  { name: 'Storm', category: 'stream', badge: 'STREAM EVENTO', icon: 'ϟ', summary: 'Cómputo continuo de baja latencia mediante topologías de spouts y bolts.', architecture: 'Los spouts emiten tuplas y los bolts transforman, agregan o persisten datos en una topología distribuida.', strength: 'Reacción evento a evento y procesamiento continuo sencillo.', limit: 'El manejo avanzado de estado y tiempo de evento es menos integrado que en Flink.', example: 'Activar alertas operacionales en segundos a partir de telemetría de infraestructura.' },
  { name: 'Samza', category: 'stream', badge: 'STREAM + KAFKA', icon: '▰', summary: 'Procesamiento paralelo de streams con estado, integrado estrechamente con Kafka y YARN.', architecture: 'Tareas consumen particiones, mantienen estado local persistente, crean checkpoints y producen nuevos streams.', strength: 'Aplicaciones orientadas a eventos con particionamiento y estado local.', limit: 'Su mejor encaje suele depender del ecosistema Kafka y de una clave de partición bien elegida.', example: 'Mantener en tiempo real el perfil de actividad de cada cliente a partir de eventos Kafka.' },
  { name: 'Drill', category: 'sql', badge: 'SCHEMA-ON-READ', icon: '⌕', summary: 'Consulta SQL distribuida sobre JSON, Parquet, CSV, HBase, MongoDB y otras fuentes.', architecture: 'Interpreta la estructura al consultar y distribuye fragmentos del plan sin exigir un esquema central previo.', strength: 'Exploración rápida de datos semiestructurados y heterogéneos.', limit: 'Schema-on-read desplaza ambigüedad, validación y costo de inferencia al momento de consulta.', example: 'Explorar logs JSON recién recibidos antes de definir un modelo definitivo.' },
  { name: 'Presto', category: 'sql', badge: 'SQL FEDERADO', icon: '⌘', summary: 'Ejecuta SQL interactivo sobre múltiples fuentes sin moverlas previamente.', architecture: 'El Coordinator analiza y planifica; los Workers ejecutan etapas en paralelo y en pipeline mediante conectores.', strength: 'Analítica federada de baja latencia para usuarios y herramientas BI.', limit: 'No es una base transaccional ni vuelve baratos los joins remotos o movimientos masivos.', example: 'Cruzar ventas en un data lake con clientes en una base relacional desde una sola consulta.' },
  { name: 'Calcite', category: 'sql', badge: 'OPTIMIZADOR', icon: '◇', summary: 'Framework de parsing, validación y optimización SQL sin almacenamiento propio.', architecture: 'Convierte SQL en álgebra relacional y aplica reglas y costos para producir planes que otro motor ejecuta.', strength: 'Incorporar optimización y federación a motores y productos de datos.', limit: 'No almacena datos ni ejecuta por sí mismo todo el pipeline distribuido.', example: 'Un motor embebido usa Calcite para decidir el orden de joins y empujar filtros al origen.' },
  { name: 'SAMOA', category: 'ml', badge: 'ML ONLINE', icon: '◎', summary: 'Abstrae algoritmos de minería y aprendizaje sobre flujos masivos.', architecture: 'Separa el algoritmo del motor subyacente para ejecutarlo sobre Storm, Flink o Samza.', strength: 'Portabilidad de algoritmos online y procesamiento incremental.', limit: 'La abstracción no elimina diferencias de garantías, estado y rendimiento entre motores.', example: 'Actualizar continuamente un clasificador de comportamiento conforme llegan nuevas observaciones.' },
]

const scenarios = {
  fraude: { title: 'Fraude bancario en tiempo real', description: 'Pagos llegan desordenados, la respuesta debe ser inmediata y se necesita estado por tarjeta.', stack: ['Kafka', 'Flink', 'Almacén de alertas'], reason: 'Kafka conserva y desacopla los eventos; Flink usa tiempo de evento, ventanas y estado; el sumidero debe soportar escrituras idempotentes.' },
  reportes: { title: 'Reporte histórico nocturno', description: 'Se procesan 40 TB una vez al día; importa el throughput más que la latencia.', stack: ['HDFS', 'Hadoop / Spark', 'Data warehouse'], reason: 'La carga es acotada y repetible. HDFS aporta localidad y réplica; Hadoop favorece batch robusto y Spark conviene si existen varias etapas o iteraciones.' },
  exploracion: { title: 'Exploración SQL multifuente', description: 'Analistas cruzan Parquet, JSON y una base relacional sin copiar todo.', stack: ['Drill / Presto', 'Conectores', 'Fuentes existentes'], reason: 'Drill es fuerte ante esquema flexible; Presto prioriza SQL federado interactivo. La decisión depende de formatos, conectores y costo de mover datos.' },
}

const filteredTools = computed(() => category.value === 'todos' ? tools : tools.filter(tool => tool.category === category.value))
const currentTool = computed(() => tools.find(tool => tool.name === activeTool.value)!)

function selectCategory(value: Category) {
  category.value = value
  const first = value === 'todos' ? tools[0] : tools.find(tool => tool.category === value)
  if (first) activeTool.value = first.name
}

</script>

<template>
  <section class="page topic12-page">
    <header class="topic12-hero">
      <div>
        <span class="eyebrow">TEMA 12 · ECOSISTEMA BIG DATA</span>
        <h1>Elige el motor por el problema,<br><em>no por su popularidad.</em></h1>
        <p>Frameworks distribuidos para almacenar, transportar, procesar, consultar y aprender de datos masivos.</p>
      </div>
      <a class="outline-button" href="/[12-1]%20BDA%20-%20Clase.pdf" target="_blank">Abrir PDF del Tema 12 ↗</a>
    </header>

    <section class="topic12-summary">
      <div><span class="eyebrow">RESUMEN EN 60 SEGUNDOS</span><h2>Una arquitectura, varias responsabilidades.</h2></div>
      <p>Hadoop y Spark cubren procesamiento masivo; Flink, Storm y Samza reaccionan a flujos; Kafka transporta y conserva eventos; Drill y Presto consultan fuentes heterogéneas; Calcite optimiza planes; SAMOA lleva aprendizaje automático a streams. La pregunta difícil no es “¿cuál es mejor?”, sino qué garantías de latencia, estado, recuperación, esquema y costo exige cada caso.</p>
      <div class="summary-rule"><span>REGLA DE EXAMEN</span><strong>Fuente → garantía → estado → tiempo → recuperación → sumidero</strong></div>
    </section>

    <section class="framework-explorer">
      <div class="section-heading"><div><span class="eyebrow">MAPA INTERACTIVO</span><h2>Explora el ecosistema por función.</h2></div></div>
      <div class="category-tabs">
        <button v-for="item in [['todos','Todos'],['batch','Batch'],['stream','Streaming'],['mensajeria','Mensajería'],['sql','SQL'],['ml','ML online']]" :key="item[0]" :class="{ active: category === item[0] }" @click="selectCategory(item[0] as Category)">{{ item[1] }}</button>
      </div>
      <div class="framework-layout">
        <div class="framework-list">
          <button v-for="tool in filteredTools" :key="tool.name" :class="{ active: activeTool === tool.name }" @click="activeTool = tool.name">
            <span>{{ tool.icon }}</span><div><strong>{{ tool.name }}</strong><small>{{ tool.badge }}</small></div><b>→</b>
          </button>
        </div>
        <Transition name="framework-card" mode="out-in">
          <article :key="currentTool.name" class="framework-detail">
            <div class="framework-title"><span>{{ currentTool.icon }}</span><div><small>{{ currentTool.badge }}</small><h2>{{ currentTool.name }}</h2></div></div>
            <p class="framework-lead">{{ currentTool.summary }}</p>
            <div class="framework-facts">
              <div><span>ARQUITECTURA</span><p>{{ currentTool.architecture }}</p></div>
              <div><span>DÓNDE DESTACA</span><p>{{ currentTool.strength }}</p></div>
              <div class="warning"><span>LÍMITE QUE EXAMINAN</span><p>{{ currentTool.limit }}</p></div>
              <div class="real-case"><span>EJEMPLO REALISTA</span><p>{{ currentTool.example }}</p></div>
            </div>
          </article>
        </Transition>
      </div>
    </section>

    <section class="architecture-lab">
      <div class="section-heading"><div><span class="eyebrow">LABORATORIO DE DECISIONES</span><h2>Diseña el pipeline según la carga.</h2></div></div>
      <div class="scenario-tabs">
        <button v-for="(item, key) in scenarios" :key="key" :class="{ active: scenario === key }" @click="scenario = key">{{ item.title }}</button>
      </div>
      <div class="scenario-stage">
        <div class="scenario-brief"><span class="eyebrow">REQUISITOS</span><h3>{{ scenarios[scenario].title }}</h3><p>{{ scenarios[scenario].description }}</p><button class="text-button" @click="pipelineStep = pipelineStep === 3 ? 0 : pipelineStep + 1">{{ pipelineStep === 3 ? 'Reiniciar flujo ↻' : 'Ejecutar siguiente etapa →' }}</button></div>
        <div class="animated-pipeline">
          <div v-for="(item, index) in scenarios[scenario].stack" :key="item" :class="{ active: pipelineStep >= index + 1 }"><span>{{ String(index + 1).padStart(2, '0') }}</span><strong>{{ item }}</strong><i v-if="index < 2">→</i></div>
        </div>
        <Transition name="reason"><p v-if="pipelineStep === 3" class="scenario-reason"><strong>Decisión defendible:</strong> {{ scenarios[scenario].reason }}</p></Transition>
      </div>
    </section>

    <section class="comparison-section">
      <span class="eyebrow">COMPARACIÓN DE ALTA UTILIDAD</span><h2>No confundas responsabilidades.</h2>
      <div class="comparison-table">
        <div class="table-row header"><span>Tecnología</span><span>Unidad dominante</span><span>Estado/recuperación</span><span>No es principalmente</span></div>
        <div class="table-row"><strong>Hadoop</strong><span>Archivo / job batch</span><span>Réplica + reejecución</span><span>Baja latencia</span></div>
        <div class="table-row"><strong>Spark</strong><span>DAG / partición</span><span>Linaje + checkpoint</span><span>“Todo siempre en RAM”</span></div>
        <div class="table-row"><strong>Flink</strong><span>Evento / ventana</span><span>Estado + checkpoint</span><span>Broker de eventos</span></div>
        <div class="table-row"><strong>Kafka</strong><span>Evento / partición</span><span>Log replicado + offset</span><span>Motor analítico completo</span></div>
        <div class="table-row"><strong>Presto / Drill</strong><span>Consulta / etapa</span><span>Reintento de consulta</span><span>Base OLTP</span></div>
      </div>
    </section>

  </section>
</template>
