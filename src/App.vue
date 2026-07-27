<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue'
import ActivitiesView from './components/ActivitiesView.vue'
import AppSidebar from './components/AppSidebar.vue'
import PracticeView from './components/PracticeView.vue'
import SummaryView from './components/SummaryView.vue'
import TopicsView from './components/TopicsView.vue'
import Topic12View from './components/Topic12View.vue'
import Topic12Activities from './components/Topic12Activities.vue'
import Topic12Quiz from './components/Topic12Quiz.vue'

type Tab = 'resumen' | 'temas' | 'tema12'
type ThemeSection = 'contenido' | 'quiz' | 'actividades'
type Simulation = 'normal' | 'fallo' | 'particion' | 'recuperado'

const tab = ref<Tab>('resumen')
const theme11Section = ref<ThemeSection>('contenido')
const theme12Section = ref<ThemeSection>('contenido')
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
    title: 'CBoC: propósito y subsistemas',
    goal: 'Explicar para qué sirve CBoC y cómo colaboran archivos, tablas y bloqueo distribuido.',
    icon: '◫',
    summary: 'CBoC —Optimización de Control Basado en Costos— sirve para procesar Big Data con escalabilidad horizontal y tolerancia a fallos. Combina tres subsistemas: archivos para almacenar y replicar, tablas para organizar y distribuir el acceso, y bloqueo para coordinar autoridad y recuperación.',
    deep: 'El punto 2 parte del límite de usar solamente un RDBMS para datos masivos y heterogéneos: NoSQL flexibiliza esquema y algunas garantías para distribuir mejor. El punto 3 añade que distribuir no basta; los fallos parciales deben ser normales. CBoC separa responsabilidades para que datos, acceso y coordinación puedan escalar y recuperarse.',
    example: 'Una plataforma almacena historiales multimedia en bloques replicados, divide su catálogo en tablas por rangos y utiliza un bloqueo distribuido para garantizar que solo un maestro coordine cambios. Si un trabajador falla, otro recupera sus réplicas; si falla el maestro, el nodo en espera adquiere autoridad.',
    trap: 'Los tres subsistemas no son alternativas ni tres copias: resuelven almacenamiento, organización y coordinación, respectivamente.',
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
  { area: 'CBoC · propósito', q: 'Una empresa afirma que CBoC sirve únicamente para elegir el plan SQL de menor costo. Según el Tema 11, ¿qué objeción es más completa?', options: ['Es correcto porque CBoC no distribuye datos.', 'El material presenta CBoC como una plataforma con archivos, tablas y bloqueo distribuido para escalar y tolerar fallos.', 'CBoC solo es un protocolo de red.', 'CBoC reemplaza cualquier aplicación empresarial.'], correct: 1, explanation: 'En el contexto del PDF, CBoC integra tres subsistemas distribuidos. Su utilidad se analiza en almacenamiento, organización, coordinación y recuperación, no solo como optimizador SQL.' },
  { area: 'CBoC · subsistemas', q: '¿Qué asignación de responsabilidades representa correctamente los tres subsistemas de CBoC?', options: ['Archivos coordinan liderazgo; tablas controlan switches; bloqueo almacena videos.', 'Archivos almacenan y replican; tablas organizan acceso; bloqueo coordina autoridad y recuperación.', 'Los tres realizan exactamente la misma función.', 'Tablas sustituyen archivos y bloqueo.'], correct: 1, explanation: 'Los subsistemas son complementarios: almacenamiento físico, organización lógica y coordinación distribuida.' },
  { area: 'CBoC · diseño', q: 'Una tabla posee tres réplicas, pero dos maestros aceptan actualizaciones incompatibles. ¿Qué componente no está cumpliendo su responsabilidad?', options: ['El subsistema de bloqueo distribuido.', 'La compresión del archivo.', 'El generador de datos de prueba.', 'El conmutador central necesariamente.'], correct: 0, explanation: 'La réplica conserva copias, pero la exclusión y autoridad corresponden al bloqueo distribuido. Sin coordinación puede ocurrir split-brain.' },
  { area: 'Figura 3 · monitoreo', q: 'En la Figura 3, ¿cuál es el objetivo principal de “(a) Alive monitoring”?', options: ['Comprimir tablas.', 'Comprobar si los procesos trabajadores continúan activos.', 'Elegir el formato de archivo.', 'Ejecutar consultas de clientes.'], correct: 1, explanation: 'El subsistema de bloqueo supervisa la vida de los trabajadores para detectar fallos y activar la secuencia de recuperación.' },
  { area: 'Figura 3 · notificación', q: 'Después de detectar que un trabajador falló, ¿qué representa la flecha “(b) Failure notification”?', options: ['El trabajador fallido se repara solo.', 'El subsistema de bloqueo informa al maestro activo de archivos o tablas.', 'El cliente adquiere el bloqueo maestro.', 'El switch replica datos.'], correct: 1, explanation: 'La detección se convierte en una decisión coordinada cuando el fallo es comunicado al maestro responsable.' },
  { area: 'Figura 3 · recuperación', q: '¿Qué actor emite “(c) Recovery instruction” y con qué finalidad?', options: ['El cliente ordena borrar réplicas.', 'El maestro activo ordena a un trabajador disponible recuperar datos y continuar.', 'El trabajador fallido elige al nuevo maestro.', 'El switch central crea una tabla.'], correct: 1, explanation: 'El maestro decide la reasignación y otro trabajador ejecuta la recuperación usando datos redundantes.' },
  { area: 'Figura 3 · síntesis', q: '¿Qué conclusión se deriva de las flechas (a), (b) y (c) de la Figura 3?', options: ['El bloqueo almacena y recupera directamente todos los bloques.', 'Detección, decisión y ejecución de recuperación están separadas entre bloqueo, maestro y trabajadores.', 'Todo fallo exige intervención manual.', 'Las réplicas eliminan la necesidad de maestro.'], correct: 1, explanation: 'La figura separa supervisión/notificación, decisión de coordinación y recuperación efectiva. Esa división evita confundir responsabilidades.' },
  { area: 'Failover maestro', q: 'El maestro activo falla. ¿Cuál secuencia respeta mejor la Figura 3?', options: ['El standby escribe antes de obtener autoridad.', 'Expira o se libera el bloqueo, el standby lo adquiere y asume como activo.', 'Todos los trabajadores se convierten en maestros.', 'Se eliminan los metadatos para evitar conflicto.'], correct: 1, explanation: 'El bloqueo exclusivo serializa la promoción: el standby solo debe actuar después de adquirir autoridad válida.' },
  { area: 'Fallo de trabajador', q: 'Un trabajador deja de responder, pero sus datos tienen réplicas. ¿Por qué la recuperación todavía requiere al maestro?', options: ['Porque las copias no deciden qué trabajador debe continuar ni qué tarea retomar.', 'Porque el maestro contiene la única copia.', 'Porque la réplica siempre está corrupta.', 'Porque el bloqueo no detecta procesos.'], correct: 0, explanation: 'La redundancia preserva información; el maestro coordina la reasignación y el punto desde el cual debe continuar el trabajo.' },
  { area: 'Redundancia', q: 'Tres copias están en servidores distintos bajo el mismo rack y la misma alimentación. ¿Qué riesgo permanece?', options: ['Ninguno, porque son tres procesos.', 'Un fallo del dominio compartido puede eliminar las tres copias simultáneamente.', 'Solo disminuye la velocidad del CPU.', 'El consenso deja de usar mayoría.'], correct: 1, explanation: 'La independencia debe evaluarse por dominios de fallo, no solo por cantidad de servidores.' },
  { area: 'Red en árbol', q: '¿Por qué CBoC coloca maestros y datos redundantes bajo diferentes switches de borde?', options: ['Para que cada switch tenga la misma dirección IP.', 'Para que la caída de un switch no desconecte simultáneamente autoridad y copias necesarias.', 'Para eliminar todo tráfico de red.', 'Para evitar replicar datos.'], correct: 1, explanation: 'La colocación cruza dominios de fallo y reduce fallos correlacionados dentro de la topología en árbol.' },
  { area: 'Partición de red', q: 'Dos ramas de red pierden comunicación. ¿Qué condición permite que una rama confirme nuevas decisiones con seguridad?', options: ['Tener el nodo con CPU más rápida.', 'Conservar un quórum y aplicar fencing frente a la rama aislada.', 'Responder primero al cliente.', 'Duplicar el timeout en ambos lados.'], correct: 1, explanation: 'El quórum evita mayorías incompatibles y el fencing impide que una autoridad obsoleta siga escribiendo.' },
  { area: 'Lease', q: '¿Por qué un lease tiene duración limitada?', options: ['Para comprimir mensajes.', 'Para que la autoridad pueda recuperarse si el propietario desaparece sin liberar el bloqueo.', 'Para garantizar que la red nunca falle.', 'Para almacenar réplicas temporales.'], correct: 1, explanation: 'La expiración evita bloqueos huérfanos, aunque debe combinarse con fencing porque un proceso pausado puede reaparecer.' },
  { area: 'Heartbeat', q: 'Un heartbeat llega tarde debido a una pausa del sistema operativo. ¿Qué inferencia es segura?', options: ['El nodo fue destruido físicamente.', 'Solo existe sospecha de fallo; la autoridad debe resolverse mediante lease, quórum y tokens.', 'El nodo conserva necesariamente el liderazgo.', 'Debe borrarse toda réplica.'], correct: 1, explanation: 'Un detector por timeout no distingue fallo, lentitud o partición. Por eso no basta para conceder autoridad de escritura.' },
  { area: 'Fencing', q: 'A posee token 18 y B, nuevo maestro, token 19. ¿Qué debe hacer el recurso si A intenta escribir?', options: ['Aceptar porque A fue maestro primero.', 'Rechazar el token 18 por ser anterior al 19.', 'Aceptar ambos y reconciliar siempre.', 'Esperar indefinidamente.'], correct: 1, explanation: 'Los fencing tokens monotónicos permiten al recurso reconocer y rechazar propietarios obsoletos.' },
  { area: 'Consenso', q: 'En cinco nodos, una decisión recibió votos de A, B y C. ¿Por qué otra mayoría no puede ser completamente independiente?', options: ['Porque toda mayoría de tres debe intersectar a la primera.', 'Porque todos los nodos tienen el mismo disco.', 'Porque solo A puede votar.', 'Porque una mayoría siempre tiene cinco miembros.'], correct: 0, explanation: 'Dos subconjuntos de tres dentro de cinco comparten al menos un nodo, preservando información entre decisiones.' },
  { area: 'RDBMS y NoSQL', q: '¿Qué requisito favorece más un RDBMS con transacciones fuertes frente a un almacén eventualmente consistente?', options: ['Actualizar el saldo de dos cuentas manteniendo una invariante.', 'Servir recomendaciones que toleran segundos de retraso.', 'Almacenar logs sin relaciones.', 'Distribuir un catálogo de solo lectura.'], correct: 0, explanation: 'Las invariantes monetarias requieren atomicidad y aislamiento claros; la obsolescencia tolerable admite garantías más débiles.' },
  { area: 'NoSQL', q: 'Una base NoSQL ofrece lectura de tus propias escrituras y convergencia eventual. ¿Qué demuestra?', options: ['Que BASE significa ausencia total de garantías.', 'Que pueden combinarse garantías de sesión con convergencia eventual.', 'Que toda operación es linealizable.', 'Que no existe replicación.'], correct: 1, explanation: 'La consistencia posee varios niveles. Debilitar linealizabilidad no implica abandonar todas las garantías.' },
  { area: 'Mash-up', q: 'Una aplicación muestra clima y tráfico en dos paneles independientes sin combinarlos. ¿Por qué puede no ser todavía un mash-up valioso?', options: ['Porque no utiliza Hadoop.', 'Porque no produce una capacidad derivada mediante composición de las fuentes.', 'Porque utiliza dos API.', 'Porque contiene una interfaz gráfica.'], correct: 1, explanation: 'El valor del mash-up aparece cuando la composición crea un servicio nuevo, como recomendar rutas según clima y congestión.' },
  { area: 'Mash-up · fallo', q: 'La API de clima no responde. ¿Qué diseño mejora la resiliencia de un mash-up de rutas?', options: ['Bloquear todo indefinidamente.', 'Definir timeout, datos cacheados y degradación explícita de la recomendación.', 'Inventar el clima actual.', 'Eliminar las otras fuentes.'], correct: 1, explanation: 'Los mash-ups heredan fallos de dependencias. Una degradación controlada mantiene servicio e informa menor confianza.' },
  { area: 'Escalabilidad', q: 'Un trabajo tarda 100 minutos y 60 corresponden a una fase serial. ¿Por qué duplicar nodos no reducirá el tiempo a la mitad?', options: ['La parte serial limita la aceleración total.', 'Los nodos adicionales eliminan la fase serial.', 'La red siempre duplica velocidad.', 'El almacenamiento no influye.'], correct: 0, explanation: 'La sección no paralelizable fija un límite superior; además aparecen coordinación y comunicación.' },
  { area: 'Skew', q: 'Una partición recibe 70% de los registros mientras nueve comparten el resto. ¿Qué ocurre al agregar trabajadores?', options: ['El trabajo queda automáticamente equilibrado.', 'La partición caliente puede dominar el tiempo total y dejar capacidad ociosa.', 'El consenso desaparece.', 'Cada registro se replica cero veces.'], correct: 1, explanation: 'El sesgo impide distribuir uniformemente la carga. Debe revisarse la clave, particionamiento o tratamiento de claves calientes.' },
  { area: 'Localidad de datos', q: '¿Qué ventaja busca ejecutar una tarea cerca de los bloques que procesa?', options: ['Reducir transferencia de grandes volúmenes por la red.', 'Eliminar toda réplica.', 'Garantizar exactly-once.', 'Sustituir el maestro.'], correct: 0, explanation: 'Mover el cómputo suele ser más barato que trasladar datasets masivos a través de la red.' },
  { area: 'Recuperación', q: 'Una tarea aplicó un cobro y falló antes de registrar su finalización. ¿Qué propiedad permite reintentarla con menor riesgo?', options: ['Idempotencia o deduplicación por identificador.', 'Mayor tamaño de archivo.', 'Menor número de logs.', 'Un heartbeat más lento.'], correct: 0, explanation: 'El resultado es incierto. Una clave idempotente evita aplicar dos veces el mismo efecto durante la recuperación.' },
  { area: 'Pruebas · estados', q: '¿Por qué probar solo la caída completa del clúster ofrece cobertura insuficiente?', options: ['Porque los fallos parciales, retrasos y reordenamientos generan estados más difíciles.', 'Porque un clúster nunca falla.', 'Porque las salidas finales no importan.', 'Porque todas las transiciones son equivalentes.'], correct: 0, explanation: 'La complejidad distribuida surge especialmente cuando unos componentes avanzan mientras otros fallan o se aíslan.' },
  { area: 'Pruebas · datos', q: 'Un generador crea el volumen correcto, pero distribuye claves uniformemente cuando producción tiene fuerte sesgo. ¿Qué puede ocultar?', options: ['Cuellos de botella, particiones calientes y tiempos de cola.', 'La existencia de CPU.', 'El formato del PDF.', 'La necesidad de usuarios.'], correct: 0, explanation: 'El volumen sin una distribución representativa no reproduce el comportamiento de particionamiento y carga real.' },
  { area: 'Observabilidad', q: 'El resultado final es correcto, pero durante cinco minutos hubo dos líderes y escrituras rechazadas. ¿Por qué debe considerarse un fallo de prueba?', options: ['Porque la trayectoria violó seguridad o disponibilidad aunque el estado final convergiera.', 'Porque todo rechazo es normal.', 'Porque solo importa el último registro.', 'Porque dos líderes mejoran throughput.'], correct: 0, explanation: 'Logs, métricas y trazas revelan violaciones transitorias que una comparación final puede ocultar.' },
  { area: 'Automatización', q: 'Puppet instala cien nodos idénticos. ¿Qué problema no resuelve por sí solo?', options: ['La repetición de configuración.', 'La elección de escenarios, oráculos y criterios de corrección.', 'La instalación automatizada.', 'La reducción de deriva.'], correct: 1, explanation: 'Infraestructura como código reproduce entornos, pero el diseño y evaluación de pruebas siguen requiriendo hipótesis y oráculos.' },
  { area: 'Checkpoint', q: '¿Qué beneficio experimental aporta iniciar una prueba desde un checkpoint conocido?', options: ['Reproducir una condición intermedia con menos preparación y variabilidad.', 'Demostrar todos los estados posibles.', 'Eliminar la necesidad de validar.', 'Garantizar que producción es idéntica.'], correct: 0, explanation: 'Los checkpoints aceleran repeticiones comparables, pero no sustituyen cobertura ni representatividad.' },
  { area: 'Síntesis CBoC', q: '¿Cuál explicación conecta mejor costo, escala y tolerancia a fallos en CBoC?', options: ['Usar más servidores siempre reduce costo y complejidad.', 'Servidores comunes permiten escalar, pero exigen distribución, redundancia y coordinación para convertir sus fallos en eventos recuperables.', 'La tolerancia elimina la necesidad de pruebas.', 'El bloqueo distribuido almacena todos los datos.'], correct: 1, explanation: 'El beneficio económico del escalado horizontal trae obligaciones: particionar, replicar, coordinar y probar recuperación.' },
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
  theme11Section.value = 'contenido'
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
    <AppSidebar
      :active-tab="tab"
      :progress="progress"
      :completed-count="completed.length"
      :topic-count="topics.length"
      @select="selectTab"
    />
    <main>
      <SummaryView
        v-if="tab === 'resumen'"
        :topics="topics"
        :completed="completed"
        :simulation="simulation"
        :logs="logs"
        :flash-question="flashcards[0]!.q"
        @select-tab="selectTab"
        @select-topic="toggleTopic"
        @simulate="runSimulation"
      />
      <div v-else-if="tab === 'temas'" class="theme-workspace">
        <nav class="theme-workspace-nav" aria-label="Módulos del Tema 11">
          <div><span>TEMA</span><strong>11</strong></div>
          <button :class="{ active: theme11Section === 'contenido' }" @click="theme11Section = 'contenido'">Contenido</button>
          <button :class="{ active: theme11Section === 'quiz' }" @click="theme11Section = 'quiz'">Quiz</button>
          <button :class="{ active: theme11Section === 'actividades' }" @click="theme11Section = 'actividades'">Actividades</button>
        </nav>
        <TopicsView v-if="theme11Section === 'contenido'" :topics="topics" :active-topic="activeTopic" :completed="completed" @select-topic="activeTopic = $event" @toggle-complete="markComplete" />
        <PracticeView v-else-if="theme11Section === 'quiz'" :flashcards="flashcards" :flash-index="flashIndex" :flash-revealed="flashRevealed" :streak="streak" :quiz-questions="quizQuestions" :quiz-started="quizStarted" :quiz-finished="quizFinished" :quiz-index="quizIndex" :quiz-answers="quizAnswers" :quiz-score="quizScore" @toggle-flash="flashRevealed = !flashRevealed" @next-flash="nextFlash" @start-quiz="startQuiz" @answer-quiz="answerQuiz" @select-quiz="quizIndex = $event" @finish-quiz="finishQuiz" />
        <ActivitiesView v-else :activities="activities" />
      </div>
      <div v-else class="theme-workspace topic12-page">
        <nav class="theme-workspace-nav topic12-nav" aria-label="Módulos del Tema 12">
          <div><span>TEMA</span><strong>12</strong></div>
          <button :class="{ active: theme12Section === 'contenido' }" @click="theme12Section = 'contenido'">Contenido</button>
          <button :class="{ active: theme12Section === 'quiz' }" @click="theme12Section = 'quiz'">Quiz</button>
          <button :class="{ active: theme12Section === 'actividades' }" @click="theme12Section = 'actividades'">Actividades</button>
        </nav>
        <Topic12View v-if="theme12Section === 'contenido'" />
        <Topic12Quiz v-else-if="theme12Section === 'quiz'" />
        <section v-else class="page">
          <header class="page-header activities-header"><div><span class="eyebrow">TEMA 12 · ACTIVIDAD DE APRENDIZAJE</span><h1>Actividades del ecosistema Big Data.</h1><p>Respuestas argumentadas sobre Hadoop, Spark, streaming, Kafka y motores SQL distribuidos.</p></div><a href="/[12-1]%20BDA%20-%20Clase.pdf" target="_blank" class="outline-button">Abrir PDF Tema 12 ↗</a></header>
          <Topic12Activities />
        </section>
      </div>
    </main>
  </div>
</template>
