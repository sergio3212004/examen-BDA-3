<script setup lang="ts">
import { computed, ref } from 'vue'

const active = ref(0)
const flowStep = ref(0)

const frameworks = [
  {
    name: 'Apache Hadoop', tag: 'ALMACENAMIENTO + BATCH', icon: '◫',
    definition: 'Ecosistema para almacenar y procesar datasets masivos de forma distribuida sobre clústeres de hardware común.',
    problem: 'Permite que un volumen mayor que la capacidad de una máquina se divida, replique y procese en paralelo.',
    components: [['HDFS', 'Divide archivos en bloques, los distribuye y replica.'], ['MapReduce', 'Ejecuta fases map, shuffle y reduce.'], ['YARN', 'Administra recursos y agenda aplicaciones.'], ['Common', 'Utilidades compartidas del ecosistema.']],
    flow: ['HDFS divide y replica bloques', 'YARN asigna contenedores', 'Map procesa bloques localmente', 'Shuffle y Reduce consolidan'],
    example: 'Una entidad financiera procesa cada noche años de transacciones para generar reportes regulatorios. La latencia de horas es aceptable; importan throughput y recuperación.',
    choose: 'Archivos grandes, procesamiento batch, reconstrucciones históricas y plataformas HDFS existentes.',
    avoid: 'Consultas interactivas, millones de archivos pequeños o respuestas evento a evento.',
    compare: 'Hadoop materializa etapas y prioriza robustez batch; Spark forma un DAG y reutiliza memoria para reducir operaciones intermedias.',
    trap: 'Hadoop no es solamente HDFS: almacenamiento, ejecución y gestión de recursos son responsabilidades distintas.',
  },
  {
    name: 'Apache Spark', tag: 'DAG + MEMORIA', icon: '✦',
    definition: 'Motor distribuido unificado para batch, SQL, streaming estructurado, machine learning y grafos.',
    problem: 'Reduce el costo de pipelines con múltiples etapas o iteraciones, evitando materializar innecesariamente cada resultado.',
    components: [['Driver', 'Construye el plan y coordina la aplicación.'], ['Executors', 'Ejecutan tareas y almacenan particiones.'], ['RDD/DataFrame', 'Abstracciones distribuidas con transformaciones.'], ['DAG Scheduler', 'Divide dependencias en etapas y tareas.']],
    flow: ['Driver crea el DAG lógico', 'Optimizador separa etapas por shuffle', 'Executors procesan particiones', 'Linaje reconstruye pérdidas'],
    example: 'Un sistema de recomendaciones itera sobre interacciones de usuarios. Cachear el conjunto reutilizado evita releerlo en cada iteración.',
    choose: 'ETL complejo, algoritmos iterativos, notebooks, SQL y equipos que necesitan una API unificada.',
    avoid: 'Eventos con estado de muy baja latencia cuando se exige tiempo de evento sofisticado y procesamiento continuo nativo.',
    compare: 'Spark sobresale en unificación y batch rápido; Flink trata el streaming como modelo principal y ofrece control más fino de estado y tiempo.',
    trap: '“En memoria” no significa “sin disco”: shuffle, spill, checkpoints y fuentes todavía usan almacenamiento.',
  },
  {
    name: 'Apache Flink', tag: 'STREAM + ESTADO', icon: '≈',
    definition: 'Motor de procesamiento distribuido para flujos limitados e ilimitados, diseñado alrededor de estado consistente y tiempo de evento.',
    problem: 'Permite calcular resultados continuos cuando los eventos llegan tarde, desordenados y deben relacionarse con historia previa.',
    components: [['JobManager', 'Coordina, agenda y recupera el job.'], ['TaskManagers', 'Ejecutan operadores paralelos.'], ['State Backend', 'Mantiene y persiste estado.'], ['Checkpoints/Watermarks', 'Recuperan coherencia y manejan progreso temporal.']],
    flow: ['Fuente recibe eventos', 'Watermark estima progreso temporal', 'Operadores actualizan estado/ventanas', 'Checkpoint coordina recuperación'],
    example: 'Uber ha documentado pipelines Kafka–Flink que agregan eventos de anuncios y coordinan checkpoints con transacciones e idempotencia.',
    choose: 'Fraude, monitoreo, ventanas, datos tardíos, CEP y aplicaciones continuas con estado.',
    avoid: 'Un reporte diario sencillo o una carga que no justifica la complejidad operacional del estado distribuido.',
    compare: 'Kafka conserva eventos; Flink los transforma manteniendo estado. No son sustitutos: normalmente trabajan juntos.',
    trap: 'Exactly-once interno no garantiza exactamente un efecto externo si el sink no es transaccional o idempotente.',
  },
  {
    name: 'Apache Kafka', tag: 'LOG DE EVENTOS', icon: '⇄',
    definition: 'Log distribuido, particionado y replicado para publicar, conservar y consumir eventos a gran escala.',
    problem: 'Desacopla aplicaciones: productores no necesitan conocer a cada consumidor y los eventos pueden reproducirse posteriormente.',
    components: [['Producer', 'Publica registros y elige partición.'], ['Topic/Partition', 'Organiza el log y define paralelismo/orden.'], ['Broker', 'Almacena particiones y réplicas.'], ['Consumer Group', 'Distribuye particiones entre consumidores.']],
    flow: ['Productor asigna clave', 'Hash dirige a una partición', 'Broker persiste y replica', 'Consumidor avanza su offset'],
    example: 'LinkedIn desarrolló Kafka para mover actividad, métricas y logs entre numerosos sistemas sin acoplar productores y consumidores.',
    choose: 'Integración de servicios, event sourcing, ingestión, replay y backbone de pipelines en tiempo real.',
    avoid: 'Consultas analíticas complejas, joins con estado o almacenamiento relacional transaccional como función principal.',
    compare: 'Kafka Streams procesa usando el log, pero Kafka broker por sí solo no reemplaza motores como Flink o Spark.',
    trap: 'El orden es por partición, no global; las réplicas aumentan disponibilidad, no el paralelismo de un consumer group.',
  },
  {
    name: 'Apache Storm', tag: 'STREAM EVENTO A EVENTO', icon: 'ϟ',
    definition: 'Motor distribuido de procesamiento continuo que modela aplicaciones como topologías permanentes.',
    problem: 'Procesa cada evento con baja latencia mediante una red de fuentes y transformaciones que permanece ejecutándose.',
    components: [['Spout', 'Lee y emite tuplas desde una fuente.'], ['Bolt', 'Filtra, agrega, transforma o escribe.'], ['Topology', 'Grafo de spouts y bolts.'], ['Nimbus/Supervisor', 'Coordina y ejecuta trabajadores.']],
    flow: ['Spout emite una tupla', 'Grouping decide el bolt destino', 'Bolts transforman/encadenan', 'Acking detecta procesamiento fallido'],
    example: 'Un centro de operaciones procesa telemetría de servidores para activar alertas en segundos mediante spouts y bolts.',
    choose: 'Topologías evento a evento, baja latencia y transformaciones continuas relativamente directas.',
    avoid: 'Aplicaciones que dependen fuertemente de tiempo de evento, estado complejo y snapshots consistentes integrados.',
    compare: 'Storm popularizó el streaming continuo; Flink integra de forma más profunda estado, watermarks, ventanas y checkpoints.',
    trap: 'Reprocesar una tupla no evita por sí solo duplicar un efecto en una base externa.',
  },
  {
    name: 'Apache Samza', tag: 'STREAM + KAFKA', icon: '▰',
    definition: 'Framework de procesamiento de streams orientado a tareas particionadas y estrechamente integrado con Kafka.',
    problem: 'Mantiene aplicaciones de eventos con estado local eficiente y recuperación a partir de logs durables.',
    components: [['Stream Task', 'Consume una partición y transforma registros.'], ['Kafka', 'Actúa como entrada, salida y respaldo de changelog.'], ['Local Store', 'Mantiene estado cerca de la tarea.'], ['Checkpoint', 'Registra posiciones para recuperar.']],
    flow: ['Kafka entrega una partición', 'Tarea actualiza estado local', 'Changelog respalda cambios', 'Salida se publica en otro stream'],
    example: 'LinkedIn creó Samza para procesar feeds de actividad y actualizar vistas o agregaciones continuas sobre su ecosistema Kafka.',
    choose: 'Arquitecturas Kafka, estado particionado por clave y aplicaciones que se benefician de almacenamiento local.',
    avoid: 'Equipos sin Kafka o casos donde se necesita un ecosistema más amplio de SQL streaming y tiempo de evento.',
    compare: 'Samza enfatiza integración Kafka y estado local; Flink ofrece un motor de flujo más general y abstracciones temporales más amplias.',
    trap: 'Si la clave de partición es incorrecta, eventos relacionados llegan a tareas distintas y el estado deja de representar la entidad.',
  },
  {
    name: 'Apache Drill', tag: 'SQL SCHEMA-ON-READ', icon: '⌕',
    definition: 'Motor SQL distribuido para explorar fuentes y formatos heterogéneos sin exigir un esquema central previo.',
    problem: 'Permite consultar rápidamente JSON, Parquet, CSV, HBase o MongoDB antes de construir un modelo rígido.',
    components: [['Drillbit', 'Nodo que acepta y ejecuta fragmentos.'], ['Storage Plugin', 'Conecta formatos y sistemas externos.'], ['Foreman', 'Coordina la consulta recibida.'], ['Schema-on-read', 'Interpreta estructura al consultar.']],
    flow: ['Recibe consulta SQL', 'Descubre esquema desde la fuente', 'Genera fragmentos distribuidos', 'Drillbits ejecutan y reúnen'],
    example: 'Un equipo de seguridad explora logs JSON recién generados, incluso cuando algunos eventos incorporan campos nuevos.',
    choose: 'Exploración ad hoc de datos semiestructurados y múltiples formatos sin preparación extensa.',
    avoid: 'Datos críticos que requieren contratos estrictos antes de cualquier consulta o cargas OLTP.',
    compare: 'Drill destaca en schema-on-read; Presto suele enfatizar federación SQL mediante un ecosistema amplio de conectores.',
    trap: 'Aplazar el esquema no elimina su complejidad: la mueve al momento de consulta y puede producir ambigüedades.',
  },
  {
    name: 'Apache Presto', tag: 'SQL FEDERADO', icon: '⌘',
    definition: 'Motor SQL distribuido para consultas analíticas interactivas sobre múltiples fuentes sin moverlas primero a un único almacén.',
    problem: 'Reduce el tiempo necesario para cruzar información de data lakes, bases relacionales y otros sistemas.',
    components: [['Coordinator', 'Analiza, optimiza y agenda consultas.'], ['Workers', 'Ejecutan etapas y operadores.'], ['Connectors', 'Traducen acceso a cada fuente.'], ['Stages/Splits', 'Dividen la consulta en trabajo paralelo.']],
    flow: ['Coordinator analiza SQL', 'Conectores obtienen metadatos', 'Plan se divide en etapas', 'Workers procesan en pipeline'],
    example: 'Un minorista cruza ventas en Parquet con clientes de una base relacional para investigar resultados del día.',
    choose: 'BI interactivo, consulta federada, data lakes y analistas que necesitan SQL sobre varias fuentes.',
    avoid: 'Transacciones fila por fila, almacenamiento primario o joins que obligan a transferir volúmenes imposibles por red.',
    compare: 'Presto ejecuta planes; Calcite es un framework que otros motores pueden usar para analizar y optimizar planes.',
    trap: '“Sin mover previamente” no significa “sin mover durante la consulta”: un join distribuido puede transferir enormes cantidades.',
  },
  {
    name: 'Apache Calcite', tag: 'FRAMEWORK OPTIMIZADOR', icon: '◇',
    definition: 'Framework embebible para parsing, validación, álgebra relacional, planificación y optimización SQL.',
    problem: 'Evita que cada motor de datos tenga que construir desde cero un frontend SQL y un optimizador basado en reglas/costos.',
    components: [['Parser/Validator', 'Comprueba sintaxis, nombres y tipos.'], ['Relational Algebra', 'Representa operaciones de forma lógica.'], ['Rules', 'Reescriben planes equivalentes.'], ['Cost Planner', 'Compara alternativas físicas.']],
    flow: ['SQL se parsea y valida', 'Se crea árbol relacional', 'Reglas generan alternativas', 'Costos eligen un plan'],
    example: 'Un producto de consultas usa Calcite para empujar filtros a la fuente y escoger el orden de joins, mientras su propio runtime ejecuta.',
    choose: 'Construcción de motores, capas SQL federadas y productos que necesitan un optimizador extensible.',
    avoid: 'Cuando se busca un almacén listo para guardar datos o un clúster que ejecute consultas por sí mismo.',
    compare: 'Calcite diseña/optimiza el plan; Presto, Drill u otro runtime distribuyen y ejecutan el trabajo.',
    trap: 'No posee almacenamiento propio: instalar Calcite no crea automáticamente una base de datos distribuida.',
  },
  {
    name: 'Apache SAMOA', tag: 'ML ONLINE DISTRIBUIDO', icon: '◎',
    definition: 'Framework para desarrollar algoritmos de machine learning y minería sobre flujos masivos independientemente del motor subyacente.',
    problem: 'Separa la lógica del algoritmo online de detalles específicos de Storm, Flink o Samza.',
    components: [['Topology API', 'Describe procesadores y conexiones.'], ['Processors', 'Implementan pasos del algoritmo.'], ['Content Events', 'Transportan observaciones y resultados.'], ['Execution Engine', 'Traduce la topología al motor elegido.']],
    flow: ['Llegan observaciones continuas', 'Modelo se actualiza incrementalmente', 'Topología distribuye aprendizaje', 'Predicciones se emiten en línea'],
    example: 'Un clasificador de comportamiento se actualiza conforme llegan eventos, sin volver a entrenar desde cero todo el histórico.',
    choose: 'Investigación y algoritmos de aprendizaje online que deben ejecutarse sobre distintos motores de streaming.',
    avoid: 'Entrenamiento batch convencional, ecosistemas con soporte limitado o cuando se requieren bibliotecas modernas específicas no disponibles.',
    compare: 'SAMOA aporta abstracción de ML; Flink/Storm/Samza aportan el runtime distribuido que realmente ejecuta la topología.',
    trap: 'Portabilidad de API no significa garantías o rendimiento idénticos entre motores.',
  },
]

const current = computed(() => frameworks[active.value]!)

function selectFramework(index: number) {
  active.value = index
  flowStep.value = 0
}

function nextFlow() {
  flowStep.value = flowStep.value === 4 ? 0 : flowStep.value + 1
}
</script>

<template>
  <section class="framework-encyclopedia">
    <div class="section-heading"><div><span class="eyebrow">GUÍA PROFUNDA · 10 FRAMEWORKS</span><h2>Cada herramienta, explicada de principio a fin.</h2></div></div>
    <div class="encyclopedia-layout">
      <nav class="encyclopedia-index">
        <button v-for="(item, index) in frameworks" :key="item.name" :class="{ active: active === index }" @click="selectFramework(index)"><span>{{ item.icon }}</span><div><strong>{{ item.name }}</strong><small>{{ item.tag }}</small></div></button>
      </nav>
      <Transition name="encyclopedia" mode="out-in">
        <article :key="current.name" class="encyclopedia-detail">
          <header><span>{{ current.icon }}</span><div><small>{{ current.tag }}</small><h2>{{ current.name }}</h2></div></header>
          <div class="definition-block"><span>¿QUÉ ES?</span><p>{{ current.definition }}</p><span>¿QUÉ PROBLEMA RESUELVE?</span><p>{{ current.problem }}</p></div>
          <h3>Componentes principales</h3>
          <div class="framework-components"><div v-for="component in current.components" :key="component[0]"><strong>{{ component[0] }}</strong><p>{{ component[1] }}</p></div></div>
          <h3>Flujo interno animado</h3>
          <div class="framework-flow"><div v-for="(item, index) in current.flow" :key="item" :class="{ active: flowStep >= index + 1 }"><span>0{{ index + 1 }}</span><strong>{{ item }}</strong><i v-if="index < 3">→</i></div></div>
          <button class="text-button flow-button" @click="nextFlow">{{ flowStep === 4 ? 'Reiniciar flujo ↻' : 'Ejecutar siguiente paso →' }}</button>
          <div class="framework-decision-grid">
            <div class="case"><span>EJEMPLO CONCRETO</span><p>{{ current.example }}</p></div>
            <div class="choose"><span>CUÁNDO ELEGIRLO</span><p>{{ current.choose }}</p></div>
            <div class="avoid"><span>CUÁNDO NO</span><p>{{ current.avoid }}</p></div>
            <div class="compare"><span>COMPARACIÓN CLAVE</span><p>{{ current.compare }}</p></div>
          </div>
          <div class="framework-exam-trap"><span>⚠ TRAMPA DE EXAMEN</span><p>{{ current.trap }}</p></div>
        </article>
      </Transition>
    </div>
  </section>
</template>
