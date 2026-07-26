<script setup lang="ts">
const phases = [
  {
    title: 'Autoaprendizaje',
    subtitle: 'Investigación individual',
    questions: [
      {
        q: '¿Cuáles son las diferencias entre procesamiento batch y streaming, y cuándo se recomienda cada uno?',
        answer: 'Batch trabaja con un conjunto acotado y prioriza throughput, costo y reproducibilidad; streaming procesa un flujo continuo y prioriza latencia, estado y orden temporal. Conviene batch para cierres contables, entrenamiento periódico o recomputaciones históricas. Streaming es apropiado para fraude, monitoreo y personalización inmediata. No son excluyentes: una empresa puede servir alertas en segundos y recalcular el histórico por la noche.',
        example: 'Uber combina eventos de Kafka y trabajos Flink para analítica publicitaria en tiempo real, mientras conserva información en Hive para reportes y análisis batch.',
        source: 'https://www.uber.com/en-SE/blog/real-time-exactly-once-ad-event-processing/',
      },
      {
        q: '¿Cómo gestionan Hadoop y Spark la tolerancia a fallos y cómo se comparan con sistemas modernos?',
        answer: 'Hadoop replica bloques en HDFS y vuelve a ejecutar tareas MapReduce fallidas. Spark puede reconstruir particiones perdidas mediante el linaje del DAG y usa checkpoints para cortar linajes costosos. Flink toma snapshots consistentes del estado y de las posiciones de entrada; Kafka replica particiones y conserva offsets. La diferencia clave es qué se recupera: datos, tarea, estado continuo o posición dentro de un log.',
        example: 'En un job Spark iterativo, perder un executor no obliga a reiniciar todo: se recalculan las particiones afectadas. En Flink, una detección de fraude necesita restaurar además el estado de cada tarjeta y la posición exacta del stream.',
      },
      {
        q: '¿Qué papel cumple Kafka en los data pipelines y cómo se integra con herramientas de análisis?',
        answer: 'Kafka funciona como log distribuido: desacopla productores y consumidores, conserva eventos para replay y permite paralelismo mediante particiones. Flink o Samza consumen esos eventos para mantener estado y producir nuevos topics; Spark los procesa en micro-lotes o streaming; Hadoop/Hive los incorpora al histórico. Kafka no sustituye al motor analítico: aporta transporte, retención y orden por partición.',
        example: 'LinkedIn describe Kafka como su “sistema circulatorio”: aplicaciones publican actividad, métricas y trazas que múltiples consumidores procesan sin depender directamente del productor.',
        source: 'https://engineering.linkedin.com/apache-kafka/how-we_re-improving-and-advancing-kafka-linkedin',
      },
      {
        q: '¿Por qué el procesamiento en memoria representa una ventaja de Spark frente a Hadoop MapReduce?',
        answer: 'MapReduce materializa etapas intermedias frecuentemente en disco; Spark conserva conjuntos reutilizados en memoria y optimiza varias transformaciones como un DAG. La ventaja crece en algoritmos iterativos, exploración y pipelines con reutilización. Sin embargo, “en memoria” no significa “sin disco”: shuffle, presión de memoria y datasets mayores que la RAM pueden provocar spill.',
        example: 'Entrenar k-means durante 30 iteraciones reutiliza el mismo conjunto de puntos. Spark puede mantenerlo cacheado; un flujo MapReduce clásico vuelve a leer y escribir resultados entre iteraciones.',
      },
      {
        q: '¿Cómo contribuyen Presto y Drill a decisiones empresariales en tiempo real?',
        answer: 'Permiten SQL distribuido de baja latencia sobre fuentes heterogéneas. Presto destaca en consulta federada mediante conectores; Drill facilita explorar JSON, Parquet y otros formatos con schema-on-read. Reducen el tiempo entre una pregunta y una respuesta, pero no convierten cualquier consulta en barata: joins remotos, falta de particionamiento y archivos pequeños todavía pueden dominar.',
        example: 'Un minorista puede cruzar ventas Parquet del data lake con inventario de una base relacional para detectar productos agotándose. Drill resulta útil si los eventos JSON cambian de estructura; Presto si se necesita federar fuentes gobernadas.',
      },
    ],
  },
  {
    title: 'Debate crítico',
    subtitle: 'Posturas defendibles',
    questions: [
      {
        q: '¿Hadoop todavía es relevante frente a Spark o Flink?',
        answer: 'Sí, aunque su papel cambió. HDFS, YARN y el ecosistema Hadoop siguen siendo útiles para almacenamiento distribuido, cargas batch robustas y plataformas existentes. Spark ofrece mejor experiencia para DAG, SQL y ML; Flink domina aplicaciones streaming con estado y tiempo de evento. Migrar solo por modernidad puede destruir una plataforma estable; conservar MapReduce para interacción o baja latencia también sería un mal ajuste.',
        example: 'Una aseguradora con petabytes en HDFS puede mantener el almacenamiento y ejecutar Spark sobre YARN. Para alertas instantáneas incorpora Kafka/Flink sin reescribir de inmediato todo el histórico.',
      },
      {
        q: '¿Qué framework es más adecuado para tiempo real: Kafka, Flink o Storm?',
        answer: 'La pregunta mezcla responsabilidades. Kafka es el log y canal durable; Flink y Storm computan. Flink suele ser más defendible cuando existen estado complejo, datos tardíos y garantías end-to-end; Storm encaja en topologías evento a evento más simples. Una solución completa normalmente combina Kafka con uno de los motores.',
        example: 'Uber documenta un pipeline Kafka–Flink con ventanas, checkpoints, transacciones y claves idempotentes para evitar duplicar efectos en analítica de anuncios.',
        source: 'https://www.uber.com/en-SE/blog/real-time-exactly-once-ad-event-processing/',
      },
      {
        q: '¿Es mejor centralizar en un framework o utilizar varias herramientas?',
        answer: 'Centralizar reduce capacitación, integraciones y operación, pero obliga a aceptar los límites de una sola abstracción. Un ecosistema especializado mejora ajuste técnico, aunque aumenta contratos, observabilidad y fallos entre sistemas. La postura prudente es usar el menor número de herramientas que cubra requisitos distintos de manera demostrable.',
        example: 'Una arquitectura razonable usa Kafka para eventos, Flink para estado en tiempo real, un lake para histórico y Presto para consulta. Añadir Storm y Samza sin una necesidad diferente solo duplicaría operación.',
      },
      {
        q: '¿Hasta qué punto la complejidad limita la adopción en pequeñas y medianas empresas?',
        answer: 'La limita cuando el costo de operar clústeres, guardias, seguridad y observabilidad supera el valor del problema. Una pyme no necesita Kafka y Flink por tener miles de eventos. Servicios administrados reducen carga operativa, pero agregan costo variable y dependencia. La decisión debe comparar volumen, latencia, impacto del fallo y capacidad del equipo.',
        example: 'Una tienda con reportes diarios puede usar una base relacional y jobs programados. Una fintech pequeña que debe detener fraude en segundos sí puede justificar un stream administrado aun con menor volumen.',
      },
      {
        q: '¿La integración de minería de datos y multimedia aporta valor en todos los contextos?',
        answer: 'No. Aporta valor cuando imágenes, audio, video o texto contienen señales relevantes que compensan etiquetado, cómputo, privacidad y error del modelo. En procesos donde los datos estructurados ya resuelven la decisión, añadir multimedia aumenta complejidad sin beneficio proporcional.',
        example: 'En radiología, modelos sobre imágenes pueden priorizar estudios sospechosos, pero requieren validación clínica y control de sesgos. Para calcular impuestos desde montos tabulares, analizar imágenes sería innecesario.',
      },
    ],
  },
  {
    title: 'Reflexión',
    subtitle: 'Síntesis aplicada',
    questions: [
      {
        q: '¿Qué postura final adoptar sobre el uso de múltiples frameworks?',
        answer: 'Adoptar arquitectura políglota con disciplina: cada herramienta debe resolver una responsabilidad distinta, tener propietario, métricas, contrato de datos y plan de recuperación. El objetivo no es acumular productos Apache, sino satisfacer garantías medibles con la menor complejidad sostenible.',
        example: 'Kafka conserva eventos, Flink calcula alertas, HDFS/S3 mantiene histórico y Presto lo consulta. Si Spark y Flink ejecutan exactamente la misma función sin justificación, existe deuda arquitectónica.',
      },
      {
        q: '¿Qué aprendizajes nuevos pueden surgir del debate?',
        answer: 'Que batch y streaming son dimensiones de una carga, no marcas; que exactly-once requiere coordinar también el sumidero; que particiones determinan orden y paralelismo; y que tolerancia a fallos no garantiza resultados correctos si el algoritmo o la clave de partición son incorrectos.',
        example: 'Uber combina checkpoints y transacciones con identificadores únicos para idempotencia y deduplicación: la garantía no depende únicamente de activar una opción en Flink.',
        source: 'https://www.uber.com/en-SE/blog/real-time-exactly-once-ad-event-processing/',
      },
      {
        q: '¿Cómo cambia la percepción sobre el rol de los frameworks en minería de datos?',
        answer: 'El framework deja de verse como “el algoritmo” y pasa a ser la infraestructura que distribuye datos, agenda trabajo, mantiene estado y recupera fallos. La calidad del conocimiento todavía depende de muestreo, variables, sesgos, evaluación y significado empresarial.',
        example: 'SAMOA puede ejecutar un clasificador online sobre varios motores, pero cambiar Storm por Flink no corrige etiquetas sesgadas ni una métrica de negocio mal definida.',
      },
      {
        q: '¿Qué dificultades aparecen al comparar frameworks?',
        answer: 'Las categorías se superponen, las métricas comerciales no son comparables y las garantías dependen de configuración y arquitectura completa. También se confunden throughput con latencia, réplica con exactamente-una-vez y API similar con semántica equivalente.',
        example: 'Dos benchmarks pueden declarar un millón de eventos por segundo, pero uno mide mensajes pequeños sin estado y otro ventanas con checkpoints. Elegir por la cifra mayor sería metodológicamente incorrecto.',
      },
      {
        q: '¿Cómo aplicar lo aprendido en una empresa, proyecto o investigación?',
        answer: 'Primero se redactan requisitos: volumen, velocidad, orden, latencia, estado, consistencia, RTO/RPO y presupuesto. Luego se construye una prueba pequeña con datos representativos, se inyectan fallos y se mide. La herramienta se elige después de observar el cuello de botella, no antes.',
        example: 'Para movilidad urbana: Kafka recibe GPS, Flink calcula congestión por ventanas y maneja datos tardíos, un lake guarda el histórico, Spark entrena predicciones y Presto permite análisis ad hoc. Cada componente tiene una razón verificable.',
      },
    ],
  },
]
</script>

<template>
  <section class="topic12-activities">
    <div class="section-heading">
      <div><span class="eyebrow">ACTIVIDAD DE APRENDIZAJE · 15 RESPUESTAS</span><h2>Respuestas argumentadas con casos reales.</h2></div>
    </div>
    <div v-for="(phase, phaseIndex) in phases" :key="phase.title" class="t12-activity-phase">
      <div class="phase-heading"><span>0{{ phaseIndex + 1 }}</span><div><span class="eyebrow">FASE {{ phaseIndex + 1 }} · {{ phase.subtitle }}</span><h2>{{ phase.title }}</h2></div></div>
      <details v-for="(item, index) in phase.questions" :key="item.q" class="activity-item">
        <summary><span>{{ String(index + 1).padStart(2, '0') }}</span><strong>{{ item.q }}</strong><b>+</b></summary>
        <div>
          <span class="eyebrow">RESPUESTA MODELO</span><p>{{ item.answer }}</p>
          <div class="t12-real-example"><span>CASO / EJEMPLO REAL</span><p>{{ item.example }}</p><a v-if="item.source" :href="item.source" target="_blank" rel="noreferrer">Consultar caso documentado ↗</a></div>
        </div>
      </details>
    </div>
  </section>
</template>
