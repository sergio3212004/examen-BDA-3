<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue'

type Tab = 'resumen' | 'temas' | 'practicar' | 'actividades'
type Simulation = 'normal' | 'fallo' | 'particion' | 'recuperado'

const tab = ref<Tab>('resumen')
const simulation = ref<Simulation>('normal')
const activeTopic = ref(0)
const flashIndex = ref(0)
const flashRevealed = ref(false)
const completed = ref<number[]>([])
const streak = ref(5)
const quizStarted = ref(false)
const quizFinished = ref(false)
const quizIndex = ref(0)
const quizAnswers = ref<(number | null)[]>(Array(20).fill(null))

const topics = [
  {
    index: '01',
    title: 'Escalabilidad y Big Data',
    goal: 'Distinguir escalado horizontal de vertical y justificar cuándo distribuir.',
    icon: '↗',
    summary: 'El escalado horizontal agrega nodos para repartir almacenamiento y cómputo. Reduce la dependencia de una máquina excepcional, pero introduce coordinación, latencia, fallos parciales y consistencia distribuida.',
    deep: 'No equivale simplemente a “poner más servidores”. El sistema debe particionar datos y trabajo, localizar recursos, redistribuir carga y continuar operando cuando una fracción de los nodos falla. El beneficio económico aparece cuando servidores relativamente comunes sustituyen a una máquina vertical cada vez más costosa.',
    example: 'Una plataforma analiza 20 TB diarios. Escalar verticalmente exige reemplazar el servidor; escalar horizontalmente permite agregar nodos y repartir archivos y tareas. Si el software no particiona el trabajo, los nuevos nodos no producen escalabilidad real.',
    trap: 'Más nodos pueden reducir el rendimiento si la coordinación, el movimiento de datos o un componente central se convierten en cuello de botella.',
  },
  {
    index: '02',
    title: 'Procesamiento a gran escala',
    goal: 'Relacionar plataformas distribuidas, macrodatos, mash-ups y generación de servicios.',
    icon: '⌘',
    summary: 'Una plataforma distribuida recopila, almacena y procesa macrodatos para extraer conocimiento. Un mash-up combina múltiples API y resultados analíticos para crear un servicio nuevo.',
    deep: 'El valor no reside solo en almacenar volumen: surge al integrar fuentes heterogéneas, ejecutar procesamiento paralelo y convertir patrones en decisiones o productos. Hadoop prioriza procesamiento por lotes cerca de los datos; Spark mantiene conjuntos de trabajo en memoria y favorece iteraciones, streaming y análisis interactivo.',
    example: 'Un servicio de movilidad combina tráfico, clima, geolocalización y eventos. El resultado no pertenece a una sola fuente: es un servicio derivado que recomienda rutas y predice demanda.',
    trap: 'Un mash-up integra servicios; no es sinónimo de data warehouse ni de una simple unión SQL.',
  },
  {
    index: '03',
    title: 'RDBMS, NoSQL y CBoC',
    goal: 'Analizar el costo de consistencia frente a flexibilidad y distribución.',
    icon: '◫',
    summary: 'Los RDBMS ofrecen estructura, transacciones y consistencia fuerte. NoSQL flexibiliza esquema y, según el modelo, consistencia para facilitar partición y escalado horizontal. CBoC reúne archivos, tablas y bloqueo distribuido.',
    deep: 'NoSQL no significa “sin consistencia”: diferentes familias ofrecen garantías distintas. La decisión se basa en patrones de acceso, necesidad transaccional, distribución geográfica y tolerancia a particiones. CBoC usa un sistema de archivos distribuido, tablas distribuidas y un subsistema de bloqueo que coordina recursos y soporta la recuperación.',
    example: 'Un catálogo global tolera que una descripción tarde segundos en converger; el débito de una cuenta requiere reglas transaccionales mucho más estrictas. Una arquitectura moderna puede usar ambos modelos.',
    trap: 'BASE no elimina toda consistencia, y ACID no obliga a ejecutar en un único servidor.',
  },
  {
    index: '04',
    title: 'Bloqueos y consenso',
    goal: 'Explicar exclusión mutua, leases, heartbeats, elección de líder y consenso.',
    icon: '⛓',
    summary: 'Un bloqueo distribuido coordina acceso exclusivo aun con fallos. Se apoya en expiración, replicación, heartbeats, elección de líder y consenso para impedir bloqueos huérfanos o propietarios simultáneos.',
    deep: 'Un timeout aislado es peligroso: un cliente lento podría creer que conserva el bloqueo. Por eso se usan leases acotados y fencing tokens crecientes, de modo que el recurso rechace órdenes de propietarios antiguos. Raft y Paxos permiten acordar un orden o valor pese a fallos, mientras exista un quórum.',
    example: 'El maestro activo posee un lease con token 41. Tras una partición, el nuevo líder recibe token 42. Aunque el maestro anterior reaparezca, el almacenamiento rechaza sus escrituras con token 41.',
    trap: 'Elegir un líder no basta: sin quórum o fencing puede ocurrir split-brain y haber dos actores convencidos de ser maestros.',
  },
  {
    index: '05',
    title: 'Tolerancia a fallos',
    goal: 'Razonar sobre failover, redundancia y dominios de fallo.',
    icon: '⬡',
    summary: 'CBoC usa maestro activo/en espera, trabajadores redundantes y supervisión. Si el activo falla, libera o expira el bloqueo; el maestro en espera lo adquiere y asume el control.',
    deep: 'La redundancia debe cruzar dominios de fallo. Dos copias bajo el mismo switch no protegen contra la caída de ese switch. Los maestros y réplicas se colocan bajo conmutadores de borde diferentes. Los trabajadores recuperan datos del proceso fallido y retoman su carga.',
    example: 'Falla un switch de borde y desconecta ocho servidores. El sistema sigue disponible porque el maestro en espera y las réplicas necesarias están en otra rama de la red.',
    trap: 'Redundancia no equivale automáticamente a tolerancia: réplicas correlacionadas por energía, rack, zona o software pueden fallar juntas.',
  },
  {
    index: '06',
    title: 'Pruebas y observabilidad',
    goal: 'Diseñar pruebas reproducibles para sistemas con enorme espacio de estados.',
    icon: '⌁',
    summary: 'La cantidad de transiciones crece de forma explosiva. Automatización, carga rápida de datos, reinicio desde estados intermedios y observabilidad reducen el costo de preparar y validar pruebas.',
    deep: 'Puppet y herramientas equivalentes describen configuración como código para reproducir entornos. Los generadores de datos deben recrear distribuciones, volumen y estados de almacenamiento. La validación observa entradas, salidas, estado interno, errores y cronología de eventos; no basta un resultado final correcto.',
    example: 'Una prueba inyecta 10 TB, detiene un trabajador durante una compactación y corta un enlace. Métricas, logs y trazas permiten comprobar recuperación, duplicados, latencia y pérdida de datos.',
    trap: 'Automatizar despliegues no automatiza el juicio sobre hipótesis, cobertura, riesgos ni interpretación de resultados.',
  },
]

const flashcards = [
  { level: 'Alta', q: '¿Por qué un heartbeat no demuestra por sí solo que un nodo conserva autoridad para escribir?', a: 'Porque solo indica comunicación reciente. Tras una partición o pausa, otro líder pudo ser elegido. Se necesita un lease válido y, preferentemente, un fencing token para que el recurso rechace escrituras obsoletas.' },
  { level: 'Alta', q: '¿Por qué 2f + 1 nodos permiten tolerar f fallos en un consenso por mayoría?', a: 'Una mayoría requiere f + 1 votos. Tras perder f nodos todavía quedan f + 1 disponibles, y dos mayorías siempre se intersectan en al menos un nodo, preservando información acordada.' },
  { level: 'Media', q: '¿Cuál es la diferencia entre alta disponibilidad y tolerancia a fallos transparente?', a: 'La alta disponibilidad puede admitir una interrupción breve durante el failover. La tolerancia transparente busca mantener el servicio sin que el cliente perciba pérdida de sesión o de operaciones confirmadas.' },
  { level: 'Alta', q: '¿Por qué tres réplicas bajo el mismo switch no cubren bien un fallo de red?', a: 'Comparten el mismo dominio de fallo. La caída del switch elimina simultáneamente todas las copias; la redundancia debe distribuirse entre racks, switches o zonas independientes.' },
]

const quizQuestions = [
  {
    area: 'Escalabilidad', q: 'Una organización duplica de 40 a 80 nodos su plataforma, pero el tiempo de respuesta empeora. El director concluye que la escalabilidad horizontal es conceptualmente inválida. ¿Qué refutación resulta técnicamente más completa?',
    options: ['La escalabilidad horizontal solo mejora almacenamiento, nunca procesamiento.', 'La adición de nodos amplía capacidad potencial, pero coordinación, redistribución, skew y secciones seriales pueden dominar el tiempo total.', 'El problema prueba que el escalado vertical siempre es superior.', 'Los servidores añadidos deben utilizar necesariamente el mismo disco compartido.'],
    correct: 1, explanation: 'Escalar horizontalmente no garantiza aceleración lineal. Los costos de comunicación, consenso, movimiento de datos, desbalance y cuellos seriales pueden superar el paralelismo ganado.',
  },
  {
    area: 'NoSQL', q: 'Ante una partición de red, un catálogo distribuido acepta actualizaciones en ambas regiones y reconcilia después. ¿Cuál afirmación evita la simplificación de que “BASE carece de consistencia”?',
    options: ['BASE prohíbe cualquier regla de integridad.', 'La consistencia eventual puede coexistir con garantías de sesión, causalidad y resolución determinista de conflictos.', 'BASE equivale a almacenar datos sin esquema ni réplica.', 'Toda lectura BASE devuelve obligatoriamente el valor más antiguo.'],
    correct: 1, explanation: 'La consistencia no es binaria. Un sistema puede debilitar la linealizabilidad y conservar otras garantías, además de converger mediante reglas explícitas.',
  },
  {
    area: 'Bloqueos', q: 'El maestro A deja de renovar su lease, se elige B, pero A continúa ejecutando por una pausa prolongada del sistema operativo. ¿Qué mecanismo impide con mayor solidez que A corrompa el recurso?',
    options: ['Aumentar indefinidamente el timeout.', 'Enviar más heartbeats desde A.', 'Asignar fencing tokens monotónicos y hacer que el recurso rechace tokens anteriores.', 'Replicar el mismo bloqueo solo en la memoria de A.'],
    correct: 2, explanation: 'El fencing token traslada la validación al recurso. Aunque A crea conservar autoridad, sus operaciones llevan un token inferior al de B y son rechazadas.',
  },
  { area: 'Tolerancia', q: 'Tres réplicas residen en servidores distintos, aunque todos dependen del mismo conmutador de borde. ¿Qué inferencia es la menos defendible?', options: ['Existe redundancia de servidor.', 'La caída de un servidor individual puede tolerarse.', 'Las copias pertenecen a dominios de fallo plenamente independientes.', 'La caída del switch puede desconectar las tres réplicas.'], correct: 2, explanation: 'Servidores distintos no implican dominios de fallo independientes si comparten red, energía, rack o zona.' },
  { area: 'Consenso', q: 'En un clúster de cinco nodos, dos quedan aislados y tres permanecen comunicados. Suponiendo consenso por mayoría y ausencia de fallos bizantinos, ¿qué conducta preserva seguridad y disponibilidad posibles?', options: ['Ambas particiones eligen líder y aceptan escrituras.', 'La partición de tres puede progresar; la de dos debe abstenerse de confirmar escrituras.', 'La partición de dos progresa porque tiene menos latencia.', 'Ninguna partición puede leer ni escribir bajo ninguna garantía.'], correct: 1, explanation: 'Tres nodos forman mayoría. La minoría debe impedir confirmaciones para evitar decisiones incompatibles; aun podría servir lecturas con garantías explícitamente más débiles.' },
  { area: 'HDFS', q: '¿Cuál carga contradice con mayor intensidad el patrón para el que un sistema como HDFS fue optimizado?', options: ['Lectura secuencial de archivos de varios gigabytes.', 'Procesamiento batch con cómputo cerca de los bloques.', 'Millones de archivos diminutos con actualizaciones aleatorias de muy baja latencia.', 'Replicación de bloques a través de diferentes nodos.'], correct: 2, explanation: 'HDFS favorece archivos grandes, throughput y acceso secuencial; los archivos diminutos presionan metadatos y las mutaciones aleatorias no encajan con su modelo.' },
  { area: 'CBoC', q: 'Si el subsistema de tablas distribuidas conserva réplicas, ¿por qué sigue siendo estructuralmente relevante el subsistema de bloqueo distribuido?', options: ['Porque replicar datos determina por sí mismo qué maestro tiene autoridad.', 'Porque la réplica aporta copias, mientras el bloqueo coordina exclusión, liderazgo y recuperación.', 'Porque el bloqueo sustituye completamente al sistema de archivos.', 'Porque evita físicamente que un servidor falle.'], correct: 1, explanation: 'Redundancia y coordinación resuelven problemas distintos. Las copias no deciden quién puede mutarlas ni cómo conmutar de maestro con seguridad.' },
  { area: 'Failover', q: 'El maestro activo falla después de enviar una escritura a una réplica, pero antes de responder al cliente. Tras el failover, ¿qué propiedad hace especialmente difícil decidir si reintentar?', options: ['La operación puede haber sido aplicada aunque el cliente no recibiera confirmación.', 'El maestro en espera siempre pierde todas las réplicas.', 'Una escritura nunca puede sobrevivir al fallo del emisor.', 'Los timeouts garantizan exactamente una ejecución.'], correct: 0, explanation: 'Es el problema del resultado incierto. Un reintento puede duplicar efectos; se requieren operaciones idempotentes, identificadores o deduplicación.' },
  { area: 'Red', q: 'CBoC distribuye maestros y datos redundantes bajo conmutadores de borde distintos. ¿Cuál es la justificación arquitectónica de mayor precisión?', options: ['Aumentar la frecuencia del procesador.', 'Evitar que una única unidad de hardware elimine simultáneamente autoridad y copias necesarias.', 'Eliminar toda latencia entre servidores.', 'Hacer innecesario el consenso.'], correct: 1, explanation: 'La colocación cruza dominios de fallo y reduce fallos correlacionados por switch; no elimina latencia ni coordinación.' },
  { area: 'Pruebas', q: 'Una prueba distribuida valida únicamente que la salida final coincide con la esperada. ¿Qué defecto epistemológico conserva?', options: ['Ninguno; el estado interno es irrelevante.', 'Puede ocultar errores transitorios, reintentos, pérdida temporal, duplicados y recuperación incorrecta.', 'Observar logs siempre modifica los datos.', 'La salida final solo sirve en bases relacionales.'], correct: 1, explanation: 'En sistemas distribuidos importa la trayectoria: estados, errores y tiempos pueden revelar violaciones que una salida final aparentemente correcta oculta.' },
  { area: 'Automatización', q: 'Puppet reconstruye cien nodos de forma reproducible. ¿Cuál conclusión excede legítimamente lo que la automatización garantiza?', options: ['Reduce deriva de configuración.', 'Disminuye trabajo manual repetitivo.', 'Demuestra que el diseño tolera todas las transiciones de fallo posibles.', 'Facilita recrear entornos de prueba.'], correct: 2, explanation: 'La automatización reproduce configuración; no demuestra cobertura, corrección del diseño ni interpretación adecuada de todos los fallos.' },
  { area: 'Entrada de datos', q: '¿Por qué reiniciar una carga masiva desde un estado intermedio es más que una mera optimización temporal?', options: ['Permite reproducir condiciones específicas y reduce variabilidad al repetir pruebas.', 'Garantiza consistencia lineal de cualquier base.', 'Elimina la necesidad de datos representativos.', 'Convierte automáticamente pruebas en producción.'], correct: 0, explanation: 'Los checkpoints aceleran y mejoran reproducibilidad: permiten volver a una condición conocida para comparar resultados y fallos.' },
  { area: 'Mash-up', q: 'Un servicio combina API meteorológica, geolocalización y predicción propia. ¿Qué condición lo caracteriza mejor como mash-up y no como simple copia de datos?', options: ['Que use obligatoriamente SQL.', 'Que produzca una capacidad derivada mediante composición de varias interfaces y fuentes.', 'Que todas las fuentes pertenezcan a la misma empresa.', 'Que no almacene ningún dato.'], correct: 1, explanation: 'El rasgo central es la composición de servicios para crear una función nueva; almacenamiento y propiedad de fuentes no lo definen.' },
  { area: 'ACID/BASE', q: 'Una plataforma usa transacciones ACID para pagos y consistencia eventual para recomendaciones. ¿Qué lectura arquitectónica es más rigurosa?', options: ['La plataforma es incoherente porque mezcla paradigmas.', 'Aplica garantías distintas según invariantes y costo de obsolescencia de cada dominio.', 'Las recomendaciones también son necesariamente ACID.', 'El pago deja de ser distribuido por usar ACID.'], correct: 1, explanation: 'La persistencia políglota es válida cuando las garantías responden a riesgos concretos; ACID y distribución no son mutuamente excluyentes.' },
  { area: 'Spark/Hadoop', q: 'Para un algoritmo iterativo que reutiliza repetidamente el mismo conjunto de trabajo, ¿por qué Spark puede aventajar a un flujo MapReduce clásico?', options: ['Porque Spark jamás usa disco.', 'Porque puede mantener datos intermedios en memoria y evitar materializarlos en cada iteración.', 'Porque no distribuye el cómputo.', 'Porque HDFS impide ejecutar tareas paralelas.'], correct: 1, explanation: 'La reutilización en memoria reduce lecturas/escrituras intermedias. No significa que Spark nunca derrame a disco ni que Hadoop carezca de paralelismo.' },
  { area: 'Consenso', q: '¿Qué propiedad de dos mayorías en un conjunto fijo sostiene la seguridad de protocolos de consenso por quórum?', options: ['Nunca comparten nodos.', 'Siempre se intersectan al menos en un nodo.', 'Contienen exactamente los mismos nodos.', 'Funcionan aunque todos los nodos fallen.'], correct: 1, explanation: 'La intersección permite que información aceptada sobreviva entre rondas y evita que dos valores incompatibles se confirmen independientemente.' },
  { area: 'Disponibilidad', q: 'Afirmar que un sistema es “altamente disponible” porque posee maestro en espera omite principalmente:', options: ['Que el failover necesita detección, autoridad segura, estado suficientemente actualizado y ruta de red utilizable.', 'Que todo maestro debe ser un servidor vertical.', 'Que disponibilidad significa cero réplicas.', 'Que un nodo en espera nunca debe recibir datos.'], correct: 0, explanation: 'La mera existencia de una copia pasiva no asegura conmutación correcta ni dentro del tiempo objetivo.' },
  { area: 'Trabajadores', q: 'Cuando falla un proceso de trabajo y otro recupera sus datos, ¿qué riesgo persiste si las tareas no son idempotentes?', options: ['La recuperación puede repetir efectos ya aplicados antes del fallo.', 'La red deja automáticamente de ser un árbol.', 'El maestro en espera se elimina.', 'Los datos redundantes se vuelven relacionales.'], correct: 0, explanation: 'El maestro puede desconocer el punto exacto alcanzado. Reejecutar sin idempotencia o deduplicación puede duplicar efectos.' },
  { area: 'Observabilidad', q: '¿Por qué sincronizar relojes y conservar identificadores de correlación mejora la validación de fallos distribuidos?', options: ['Porque evita todos los fallos.', 'Porque permite reconstruir causalidad aproximada entre eventos dispersos y seguir una solicitud.', 'Porque reemplaza las métricas.', 'Porque convierte logs en bloqueos.'], correct: 1, explanation: 'La cronología y correlación permiten reconstruir secuencias entre nodos; aun así, el reloj físico no prueba causalidad perfecta.' },
  { area: 'Síntesis', q: '¿Cuál formulación representa mejor la relación entre escala y tolerancia a fallos descrita en el tema?', options: ['Al aumentar nodos, la probabilidad de cualquier fallo desaparece.', 'Más nodos elevan la probabilidad de fallos parciales, por lo que el sistema debe convertirlos en eventos ordinarios y recuperables.', 'Escalar horizontalmente hace innecesarias las pruebas.', 'La redundancia elimina toda complejidad operacional.'], correct: 1, explanation: 'A gran escala algún componente fallará con frecuencia. La arquitectura debe aislar, detectar y recuperar sin caída global.' },
]

const activities = [
  {
    phase: 'Fase 1 · Autoaprendizaje',
    items: [
      ['¿Cuál es la relación entre la nube y Hadoop o Spark?', 'La nube proporciona recursos elásticos —cómputo, red y almacenamiento— y Hadoop/Spark proporcionan modelos de ejecución distribuida. Hadoop divide datos en HDFS y mueve tareas hacia ellos; Spark forma un DAG de transformaciones y aprovecha memoria para iteración. En la nube pueden crearse clústeres bajo demanda, separar cómputo de almacenamiento y pagar por uso. El beneficio es elasticidad; los riesgos son costo de transferencia, dependencia del proveedor y rendimiento variable.'],
      ['¿Qué diferencias fundamentales existen entre ACID y BASE?', 'ACID prioriza atomicidad, consistencia definida por reglas, aislamiento y durabilidad por transacción. BASE acepta disponibilidad básica, estado flexible y convergencia eventual para operar distribuido. No son enemigos absolutos: una aplicación puede usar ACID para pagos y convergencia eventual para recomendaciones. El impacto de diseño aparece en conflictos, experiencia de usuario, compensaciones y elección de almacén.'],
      ['¿Cómo funcionan Paxos o Raft y por qué importan?', 'Ambos logran acuerdo entre nodos pese a fallos. Paxos separa proponentes, aceptadores y aprendices y usa rondas numeradas; Raft facilita comprensión mediante líder, log replicado y términos. Una entrada se confirma cuando alcanza mayoría. La intersección de mayorías evita decisiones incompatibles. Son esenciales para elección de líder, metadatos, bloqueos y configuración consistente.'],
      ['¿Qué papel cumple HDFS y cómo optimiza el acceso?', 'HDFS fragmenta archivos grandes en bloques, los distribuye y replica. Un NameNode conserva metadatos y DataNodes almacenan bloques. La localidad de datos lleva el cómputo hacia el nodo que contiene el bloque, reduciendo tráfico. Optimiza lecturas secuenciales de gran volumen, no archivos pequeños ni actualizaciones aleatorias de baja latencia.'],
      ['Analice casos de Google, Amazon o Netflix.', 'Google popularizó MapReduce, GFS y Bigtable para indexación y servicios globales; obtiene paralelismo y recuperación automática. Amazon usa almacenamiento y bases distribuidas como S3 y DynamoDB para elasticidad y disponibilidad a gran escala. Netflix procesa telemetría y comportamiento sobre infraestructura cloud para personalización, experimentación y operación resiliente. En los tres casos, el beneficio central es convertir volumen y distribución en decisiones y servicios confiables.'],
    ],
  },
  {
    phase: 'Fase 2 · Debate crítico',
    items: [
      ['¿Es más importante consistencia o disponibilidad?', 'Depende del daño de una lectura obsoleta o de una interrupción. En transferencias se privilegia consistencia; en un catálogo o recomendaciones, disponibilidad con convergencia puede ser aceptable. La postura defendible no elige universalmente: clasifica operaciones, define garantías por dominio y diseña degradación controlada durante particiones.'],
      ['¿NoSQL es evolución o limitación frente a lo relacional?', 'Es una evolución del repertorio, no un reemplazo universal. Amplía opciones para documentos, grafos, clave-valor y distribución masiva, pero puede trasladar joins, validación y conflictos a la aplicación. Es limitación cuando se elige por moda para un dominio transaccional; es evolución cuando el modelo coincide con accesos y escala.'],
      ['¿El escalado horizontal siempre supera al vertical?', 'No. Vertical es más simple, reduce coordinación y puede ser óptimo hasta cierto volumen. Horizontal aporta elasticidad y tolerancia, pero añade partición, replicación y operación compleja. Se justifica cuando el techo físico, el crecimiento o la disponibilidad compensan ese costo. Muchas arquitecturas combinan ambos.'],
      ['¿La tolerancia a fallos justifica la complejidad?', 'Cuando el costo esperado del fallo supera el costo de redundancia y operación. Un sistema clínico o financiero exige mayor inversión que una herramienta interna recuperable. La decisión debe basarse en RTO, RPO, impacto económico, probabilidad de fallos correlacionados y capacidad del equipo.'],
      ['¿Puppet reemplaza completamente la intervención humana?', 'No. Automatiza estados repetibles, instalación y corrección de deriva, pero humanos definen políticas, revisan cambios, responden a casos no previstos e interpretan riesgos. La meta es mover trabajo de ejecución manual hacia diseño, verificación y mejora.'],
    ],
  },
  {
    phase: 'Fase 3 · Reflexión',
    items: [
      ['¿Qué postura final adoptar sobre consistencia, disponibilidad y escalabilidad?', 'El equilibrio debe definirse por operación y riesgo. Se recomienda consistencia fuerte en invariantes críticas, disponibilidad con garantías más débiles en funciones tolerantes a obsolescencia, y escalabilidad guiada por mediciones. No existe una combinación máxima de las tres bajo cualquier fallo.'],
      ['¿Qué argumentos son más sólidos y por qué?', 'Los argumentos basados en escenarios concretos, métricas y consecuencias son más sólidos que afirmaciones universales. CAP, quórums y dominios de fallo explican límites; RTO/RPO y costo de inconsistencia conectan esos límites con decisiones reales.'],
      ['¿Cómo cambia la comprensión tras el debate?', 'Big Data deja de significar solo volumen: implica distribución, fallos parciales, modelos de consistencia, automatización y observabilidad. Cada mejora de escala crea nuevas obligaciones de coordinación y prueba.'],
      ['¿Qué resulta más relevante profesionalmente?', 'Diseñar con fallos como condición normal, escoger garantías según el negocio, automatizar entornos reproducibles y construir observabilidad desde el inicio. Estas competencias aplican a datos, backend, nube y SRE.'],
      ['¿Qué temas requieren mayor profundidad?', 'Fencing tokens, consenso formal, consistencia causal, CRDT, pruebas de caos, sesgo de particiones, seguridad de clústeres y análisis de costo cloud. Son extensiones naturales porque resuelven límites apenas introducidos en la clase.'],
    ],
  },
]

const logs = computed(() => {
  if (simulation.value === 'fallo') return ['ALERT · Heartbeat del maestro expiró', 'LOCK · Lease exclusivo liberado', 'ELECTION · Nodo 02 solicita mayoría', 'SUCCESS · Nodo 02 promovido a maestro activo']
  if (simulation.value === 'particion') return ['WARN · Switch de borde A inaccesible', 'QUORUM · Rama B conserva 3/5 votos', 'FENCE · Escrituras del maestro aislado rechazadas', 'RECOVERY · Réplicas sirven solicitudes']
  if (simulation.value === 'recuperado') return ['SYNC · Nodo 01 se reincorpora como seguidor', 'REPLAY · Registro actualizado desde índice 1842', 'HEALTH · Redundancia restaurada', 'STATUS · Clúster estable']
  return ['SYNC · Réplicas al día', 'HEARTBEAT · Maestro activo', 'QUORUM · 5/5 nodos conectados', 'STATUS · Esperando escenario']
})

const progress = computed(() => Math.round((completed.value.length / topics.length) * 100))
const quizScore = computed(() => quizQuestions.reduce((score, question, index) => score + (quizAnswers.value[index] === question.correct ? 1 : 0), 0))

function selectTab(value: Tab) {
  tab.value = value
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

function toggleTopic(index: number) {
  activeTopic.value = index
  tab.value = 'temas'
}

function markComplete(index: number) {
  completed.value = completed.value.includes(index)
    ? completed.value.filter((item) => item !== index)
    : [...completed.value, index]
}

function runSimulation(state: Simulation) {
  simulation.value = state
}

function nextFlash(knew: boolean) {
  if (knew) streak.value += 1
  else streak.value = 0
  flashIndex.value = (flashIndex.value + 1) % flashcards.length
  flashRevealed.value = false
}

function startQuiz() {
  quizAnswers.value = Array(quizQuestions.length).fill(null)
  quizIndex.value = 0
  quizFinished.value = false
  quizStarted.value = true
}

function answerQuiz(option: number) {
  if (quizFinished.value) return
  quizAnswers.value[quizIndex.value] = option
}

function finishQuiz() {
  quizFinished.value = true
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

onMounted(() => {
  const saved = localStorage.getItem('bda-study-progress')
  if (saved) completed.value = JSON.parse(saved)
})

watch(completed, (value) => localStorage.setItem('bda-study-progress', JSON.stringify(value)), { deep: true })
</script>

<template>
  <div class="app-shell">
    <aside class="sidebar">
      <button class="brand" @click="selectTab('resumen')" aria-label="Ir al resumen">
        <span class="brand-icon">▤</span>
        <span><small>BDA</small><strong>STUDY LAB</strong></span>
      </button>
      <nav class="main-nav" aria-label="Navegación principal">
        <button v-for="item in [
          ['resumen', '⌂', 'Resumen'],
          ['temas', '▦', 'Temas'],
          ['practicar', '✎', 'Practicar'],
          ['actividades', '✓', 'Actividades']
        ]" :key="item[0]" :class="{ active: tab === item[0] }" @click="selectTab(item[0] as Tab)">
          <span>{{ item[1] }}</span>{{ item[2] }}
        </button>
      </nav>
      <div class="side-progress">
        <div><span class="eyebrow">PROGRESO GENERAL</span><strong>{{ progress }}%</strong></div>
        <div class="progress"><span :style="{ width: progress + '%' }"></span></div>
        <small>{{ completed.length }} de {{ topics.length }} módulos completados</small>
      </div>
    </aside>

    <main>
      <section v-if="tab === 'resumen'" class="page">
        <header class="hero">
          <div>
            <span class="eyebrow">LAB SESSION · BASES DE DATOS AVANZADAS</span>
            <h1>Sistemas distribuidos,<br><em>sin memorizar a ciegas.</em></h1>
            <p>Comprende los mecanismos, provoca fallos y practica cómo justificar cada decisión técnica.</p>
          </div>
          <div class="hero-action">
            <div class="objective"><span class="eyebrow">OBJETIVO DE SESIÓN</span><strong>Explicar un failover sin confundir disponibilidad, consenso y consistencia.</strong></div>
            <button class="primary" @click="toggleTopic(4)">Continuar estudiando <span>→</span></button>
          </div>
        </header>

        <section class="simulator">
          <div class="section-heading">
            <div><span class="eyebrow">SYSTEM OBSERVABILITY SIMULATOR</span><h2>Provoca el fallo. Observa la coordinación.</h2></div>
            <span class="status" :class="simulation">{{ simulation === 'normal' ? 'CLUSTER: ESTABLE' : simulation === 'recuperado' ? 'CLUSTER: RECUPERADO' : 'CLUSTER: DEGRADADO' }}</span>
          </div>
          <div class="sim-controls">
            <button @click="runSimulation('fallo')">Simular fallo maestro</button>
            <button @click="runSimulation('particion')">Partición de red</button>
            <button @click="runSimulation('recuperado')">Recuperar</button>
            <button @click="runSimulation('normal')">Restablecer</button>
          </div>
          <div class="sim-grid">
            <div class="network" :class="simulation">
              <div class="client node"><b>CLIENTES</b><span>Solicitudes</span></div>
              <div class="network-line line-a"></div>
              <div class="network-line line-b"></div>
              <div class="masters">
                <div class="node master" :class="{ failed: simulation === 'fallo' || simulation === 'particion' }"><span class="pulse"></span><b>NODO 01</b><small>{{ simulation === 'fallo' ? 'FALLO' : simulation === 'particion' ? 'AISLADO' : 'MAESTRO' }}</small></div>
                <div class="node standby" :class="{ promoted: simulation !== 'normal' }"><b>NODO 02</b><small>{{ simulation === 'normal' ? 'EN ESPERA' : 'NUEVO MAESTRO' }}</small></div>
              </div>
              <div class="replicas">
                <div v-for="n in 3" :key="n" class="node replica"><span>▤</span><small>RÉPLICA {{ n }}</small></div>
              </div>
              <span class="network-label">BLOQUEO EXCLUSIVO + HEARTBEAT + QUÓRUM</span>
            </div>
            <div class="terminal">
              <div class="terminal-head"><span>●</span> SYSTEM_LOG_OUTPUT</div>
              <p v-for="(log, i) in logs" :key="log"><time>[12:05:0{{ i + 1 }}]</time> {{ log }}</p>
              <div class="terminal-question"><span>TRAMPA DE EXAMEN</span>¿Por qué un timeout no basta para evitar dos maestros?</div>
            </div>
          </div>
          <div class="simulation-explain">
            <strong>{{ simulation === 'normal' ? 'Estado normal' : simulation === 'fallo' ? 'Conmutación por error' : simulation === 'particion' ? 'Partición y split-brain' : 'Reincorporación segura' }}</strong>
            <p v-if="simulation === 'normal'">El maestro activo conserva el bloqueo exclusivo y emite heartbeats. El nodo en espera replica estado, pero no tiene autoridad para escribir.</p>
            <p v-else-if="simulation === 'fallo'">Al expirar el lease, se libera la autoridad anterior. El nodo en espera obtiene mayoría, adquiere el bloqueo y continúa el servicio.</p>
            <p v-else-if="simulation === 'particion'">La rama con mayoría elige líder. La rama minoritaria debe rechazar escrituras; los fencing tokens impiden que un maestro aislado opere con autoridad obsoleta.</p>
            <p v-else>El nodo recuperado no vuelve como maestro: primero sincroniza el registro y se reincorpora como seguidor para evitar sobrescribir decisiones confirmadas.</p>
          </div>
        </section>

        <section>
          <div class="section-heading"><div><span class="eyebrow">RUTA DE DOMINIO</span><h2>Seis módulos, una sola arquitectura mental.</h2></div><button class="text-button" @click="selectTab('temas')">Ver todos →</button></div>
          <div class="module-grid">
            <button v-for="(topic, i) in topics" :key="topic.title" class="module-card" @click="toggleTopic(i)">
              <span class="module-number">{{ topic.index }}</span><span class="module-icon">{{ topic.icon }}</span>
              <h3>{{ topic.title }}</h3><p>{{ topic.goal }}</p>
              <div><span>{{ completed.includes(i) ? 'COMPLETADO' : i === 4 ? 'EN CURSO' : 'POR ESTUDIAR' }}</span><b>→</b></div>
            </button>
          </div>
        </section>

        <section class="dashboard-bottom">
          <div class="quick-card">
            <span class="eyebrow">RECUPERACIÓN ACTIVA</span><h2>¿Puedes explicarlo sin mirar?</h2>
            <p>{{ flashcards[0].q }}</p><button class="primary" @click="selectTab('practicar')">Practicar ahora →</button>
          </div>
          <button class="activities-card" @click="selectTab('actividades')">
            <span class="eyebrow">APARTADO SOLICITADO</span><strong>15 actividades resueltas</strong>
            <p>Autoaprendizaje, debate crítico y reflexión con respuestas modelo.</p><span>ABRIR GUÍAS →</span>
          </button>
        </section>
      </section>

      <section v-else-if="tab === 'temas'" class="page">
        <header class="page-header"><span class="eyebrow">REPOSITORIO CONCEPTUAL · 06 MÓDULOS</span><h1>Entender antes de memorizar.</h1><p>Cada tema conecta mecanismo, aplicación, caso límite y trampa de examen.</p></header>
        <div class="topic-layout">
          <aside class="topic-index">
            <button v-for="(topic, i) in topics" :key="topic.title" :class="{ active: activeTopic === i }" @click="activeTopic = i"><span>{{ topic.index }}</span>{{ topic.title }}</button>
          </aside>
          <article class="reader">
            <div class="reader-heading"><div><span class="eyebrow">MÓDULO {{ topics[activeTopic].index }} · DIFICULTAD ALTA</span><h2>{{ topics[activeTopic].title }}</h2><p>{{ topics[activeTopic].goal }}</p></div><button class="complete" :class="{ done: completed.includes(activeTopic) }" @click="markComplete(activeTopic)">{{ completed.includes(activeTopic) ? '✓ Completado' : 'Marcar completado' }}</button></div>
            <div class="concept-block central"><span>IDEA CENTRAL</span><p>{{ topics[activeTopic].summary }}</p></div>
            <div class="reader-columns">
              <div class="concept-block"><span>CÓMO FUNCIONA</span><p>{{ topics[activeTopic].deep }}</p></div>
              <div class="concept-block example"><span>EJEMPLO APLICADO</span><p>{{ topics[activeTopic].example }}</p></div>
            </div>
            <div class="concept-block trap"><span>⚠ TRAMPA DE EXAMEN</span><p>{{ topics[activeTopic].trap }}</p></div>
            <div class="analysis-pattern">
              <span class="eyebrow">PLANTILLA PARA RESPONDER AL PROFESOR</span>
              <h3>Contexto → mecanismo → trade-off → consecuencia</h3>
              <ol><li>Define la condición concreta del escenario.</li><li>Nombra el mecanismo distribuido que interviene.</li><li>Explica qué garantía se gana y cuál se debilita.</li><li>Cierra con una consecuencia observable o una condición límite.</li></ol>
            </div>
            <div class="reader-nav"><button :disabled="activeTopic === 0" @click="activeTopic--">← Anterior</button><span>{{ activeTopic + 1 }} / {{ topics.length }}</span><button :disabled="activeTopic === topics.length - 1" @click="activeTopic++">Siguiente →</button></div>
          </article>
        </div>
      </section>

      <section v-else-if="tab === 'practicar'" class="page">
        <header class="page-header"><span class="eyebrow">ACTIVE RECALL · DIFICULTAD ADAPTATIVA</span><h1>Practica el razonamiento,<br>no el reconocimiento.</h1><p>Intenta responder en voz alta antes de revelar la solución.</p></header>
        <div class="practice-layout">
          <div>
            <div class="practice-meta"><span>FLASHCARD {{ flashIndex + 1 }} / {{ flashcards.length }}</span><span>RACHA: {{ streak }}</span><span>{{ flashcards[flashIndex].level }}</span></div>
            <button class="flashcard" :class="{ revealed: flashRevealed }" @click="flashRevealed = !flashRevealed">
              <div v-if="!flashRevealed"><span class="eyebrow">PREGUNTA TÉCNICA</span><h2>{{ flashcards[flashIndex].q }}</h2><small>TOCA PARA REVELAR ↻</small></div>
              <div v-else><span class="eyebrow">RESPUESTA MODELO</span><p>{{ flashcards[flashIndex].a }}</p><small>TOCA PARA VOLVER ↻</small></div>
            </button>
            <div v-if="flashRevealed" class="recall-actions"><button @click="nextFlash(true)">✓ Lo sabía</button><button @click="nextFlash(false)">↻ Necesito repasar</button></div>
          </div>
          <aside class="practice-guide"><span class="eyebrow">MÉTODO</span><h3>Cómo usar estas tarjetas</h3><ol><li>Formula una respuesta completa.</li><li>Incluye mecanismo y trade-off.</li><li>Revela y detecta la omisión.</li><li>Marca con honestidad.</li></ol></aside>
        </div>
        <section class="quiz-section">
          <div v-if="!quizStarted" class="quiz-intro">
            <div>
              <span class="eyebrow">SIMULACRO DE ALTA DIFICULTAD</span>
              <h2>20 preguntas para detectar si realmente entendiste.</h2>
              <p>Las alternativas incorrectas son técnicamente plausibles. Lee los calificadores, identifica el dominio de fallo y decide qué garantía está realmente en juego.</p>
              <ul><li>20 casos de análisis</li><li>Una sola alternativa más defendible</li><li>Explicación razonada al finalizar</li></ul>
            </div>
            <button class="primary quiz-start" @click="startQuiz">Iniciar simulacro <span>→</span></button>
          </div>

          <div v-else-if="!quizFinished" class="quiz-runner">
            <div class="quiz-top">
              <div><span class="eyebrow">SIMULACRO · PREGUNTA {{ quizIndex + 1 }} DE {{ quizQuestions.length }}</span><h2>{{ quizQuestions[quizIndex].area }}</h2></div>
              <strong>{{ Math.round(((quizIndex + 1) / quizQuestions.length) * 100) }}%</strong>
            </div>
            <div class="quiz-progress"><span :style="{ width: ((quizIndex + 1) / quizQuestions.length) * 100 + '%' }"></span></div>
            <article class="quiz-card">
              <span class="difficulty">DIFICULTAD ALTA · ANÁLISIS</span>
              <h3>{{ quizQuestions[quizIndex].q }}</h3>
              <div class="quiz-options">
                <button v-for="(option, oi) in quizQuestions[quizIndex].options" :key="option" :class="{ selected: quizAnswers[quizIndex] === oi }" @click="answerQuiz(oi)">
                  <span>{{ String.fromCharCode(65 + oi) }}</span><strong>{{ option }}</strong>
                </button>
              </div>
            </article>
            <div class="quiz-nav">
              <button :disabled="quizIndex === 0" @click="quizIndex--">← Anterior</button>
              <div class="quiz-dots"><button v-for="(_, qi) in quizQuestions" :key="qi" :class="{ current: quizIndex === qi, answered: quizAnswers[qi] !== null }" :aria-label="'Ir a pregunta ' + (qi + 1)" @click="quizIndex = qi"></button></div>
              <button v-if="quizIndex < quizQuestions.length - 1" :disabled="quizAnswers[quizIndex] === null" @click="quizIndex++">Siguiente →</button>
              <button v-else class="finish" :disabled="quizAnswers.some(answer => answer === null)" @click="finishQuiz">Finalizar</button>
            </div>
            <p v-if="quizAnswers.some(answer => answer === null) && quizIndex === quizQuestions.length - 1" class="quiz-warning">Debes responder las 20 preguntas antes de finalizar.</p>
          </div>

          <div v-else class="quiz-results">
            <header>
              <span class="eyebrow">RESULTADO DEL SIMULACRO</span>
              <div class="score-ring"><strong>{{ quizScore }}</strong><span>/ 20</span></div>
              <h2>{{ quizScore >= 17 ? 'Dominio sólido' : quizScore >= 13 ? 'Buen razonamiento, con fisuras' : quizScore >= 10 ? 'Comprensión todavía inestable' : 'Necesitas reconstruir los fundamentos' }}</h2>
              <p>{{ Math.round((quizScore / quizQuestions.length) * 100) }}% de precisión. Revisa especialmente las preguntas falladas y explica por qué cada distractor no es defendible.</p>
              <button class="outline-button" @click="startQuiz">Reintentar simulacro ↻</button>
            </header>
            <div class="result-list">
              <details v-for="(question, i) in quizQuestions" :key="question.q" :class="{ correct: quizAnswers[i] === question.correct, wrong: quizAnswers[i] !== question.correct }">
                <summary><span>{{ quizAnswers[i] === question.correct ? '✓' : '×' }}</span><strong>{{ i + 1 }}. {{ question.q }}</strong><b>+</b></summary>
                <div>
                  <p><span>TU RESPUESTA</span>{{ question.options[quizAnswers[i] ?? 0] }}</p>
                  <p><span>RESPUESTA CORRECTA</span>{{ question.options[question.correct] }}</p>
                  <p><span>RAZONAMIENTO</span>{{ question.explanation }}</p>
                </div>
              </details>
            </div>
          </div>
        </section>
      </section>

      <section v-else class="page">
        <header class="page-header activities-header"><div><span class="eyebrow">ACTIVIDAD DE APRENDIZAJE · 15 RESPUESTAS</span><h1>Actividades resueltas.</h1><p>Respuestas modelo argumentadas. Úsalas para contrastar tu razonamiento, no para copiar sin comprender.</p></div><a href="/[11-1]%20BDA%20-%20Clase.pdf" target="_blank" class="outline-button">Abrir PDF fuente ↗</a></header>
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
    </main>
  </div>
</template>
