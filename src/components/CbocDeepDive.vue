<script setup lang="ts">
import { computed, ref } from 'vue'

type Layer = 'files' | 'tables' | 'locks'
type Failure = 'master' | 'worker' | 'network'

const activeLayer = ref<Layer>('files')
const failure = ref<Failure>('master')
const step = ref(0)
const figureStep = ref(0)

const figureSteps = [
  { label: 'Estado normal', detail: 'El maestro activo de archivos/tablas posee el bloqueo exclusivo. El standby replica estado, pero todavía no tiene autoridad.' },
  { label: '(a) Supervisión', detail: 'Los servidores de bloqueo comprueban que los procesos trabajadores continúen activos mediante señales periódicas.' },
  { label: 'Fallo detectado', detail: 'Un trabajador deja de responder. Tener datos redundantes evita pérdida, pero aún se necesita coordinar quién continúa.' },
  { label: '(b) Notificación', detail: 'El subsistema de bloqueo comunica el fallo al maestro activo de archivos o tablas.' },
  { label: '(c) Recuperación', detail: 'El maestro selecciona otro trabajador y le ordena recuperar las réplicas y retomar la carga del proceso fallido.' },
  { label: 'Failover del maestro', detail: 'Si falla el propio maestro, su bloqueo expira o se libera; el standby lo adquiere mediante el servicio respaldado por consenso.' },
  { label: 'Servicio restaurado', detail: 'Existe nuevamente un solo maestro autorizado y los trabajadores disponibles gestionan los datos redundantes.' },
]

const layers = {
  files: {
    number: '01',
    icon: '▤',
    name: 'Archivos distribuidos',
    role: 'Almacenamiento físico distribuido',
    purpose: 'Divide archivos grandes en bloques, los reparte entre servidores y conserva copias redundantes.',
    helps: 'Aumenta capacidad y throughput; si un servidor pierde un bloque, otra réplica permite leerlo y reconstruirlo.',
    example: 'Un video de 12 GB se divide en bloques. Tres copias quedan en servidores y ramas de red diferentes; la caída de un nodo no elimina el archivo.',
    misconception: 'No decide quién puede modificar metadatos ni quién es el maestro.',
  },
  tables: {
    number: '02',
    icon: '▦',
    name: 'Tablas distribuidas',
    role: 'Organización y acceso lógico',
    purpose: 'Organiza registros en particiones o rangos distribuidos y dirige cada consulta al proceso que administra esos datos.',
    helps: 'Permite consultar y actualizar una tabla mayor que un servidor, balancear carga y reasignar particiones cuando falla un trabajador.',
    example: 'Una tabla con mil millones de clientes se divide por rangos. Si falla el trabajador del rango M–R, otro recupera sus réplicas y continúa.',
    misconception: 'Tener réplicas no determina por sí solo qué proceso posee autoridad para escribir.',
  },
  locks: {
    number: '03',
    icon: '⛓',
    name: 'Bloqueo distribuido',
    role: 'Coordinación, autoridad y supervisión',
    purpose: 'Mantiene exclusión mutua, supervisa maestros y trabajadores, y coordina el failover mediante bloqueo exclusivo y consenso.',
    helps: 'Evita propietarios simultáneos, detecta fallos y permite que un maestro en espera asuma control de manera coordinada.',
    example: 'El maestro A pierde su lease. El maestro B obtiene el bloqueo exclusivo y ordena la recuperación; A ya no debe aceptar escrituras.',
    misconception: 'No almacena el archivo ni sustituye la tabla: coordina a quienes los administran.',
  },
}

const failures = {
  master: {
    title: 'Falla el maestro',
    stages: [
      ['01', 'Detección', 'Expira el heartbeat o lease del maestro activo.'],
      ['02', 'Consenso', 'El servicio de bloqueo confirma la nueva autoridad.'],
      ['03', 'Failover', 'El maestro en espera adquiere el bloqueo exclusivo.'],
      ['04', 'Continuidad', 'El nuevo maestro retoma metadatos y coordina trabajadores.'],
    ],
    result: 'Los datos permanecen en archivos/tablas; lo que cambia es el proceso autorizado para coordinarlos.',
  },
  worker: {
    title: 'Falla un trabajador',
    stages: [
      ['01', 'Supervisión', 'El bloqueo distribuido detecta que dejó de responder.'],
      ['02', 'Notificación', 'El fallo se comunica al maestro activo.'],
      ['03', 'Reasignación', 'El maestro elige otro proceso de trabajo.'],
      ['04', 'Recuperación', 'El sustituto carga réplicas y retoma la tarea.'],
    ],
    result: 'La redundancia conserva los datos; la coordinación decide quién continúa procesándolos.',
  },
  network: {
    title: 'Falla un switch de borde',
    stages: [
      ['01', 'Aislamiento', 'Varios servidores pierden conectividad al mismo tiempo.'],
      ['02', 'Dominio de fallo', 'Maestros y copias están ubicados bajo switches distintos.'],
      ['03', 'Ruta válida', 'La rama disponible conserva autoridad y datos suficientes.'],
      ['04', 'Servicio', 'Las solicitudes continúan con capacidad degradada.'],
    ],
    result: 'Distribuir copias entre servidores no basta: también deben cruzar racks, switches o zonas.',
  },
}

const currentLayer = computed(() => layers[activeLayer.value])
const currentFailure = computed(() => failures[failure.value])

function runFailure(value: Failure) {
  failure.value = value
  step.value = 0
}

function advance() {
  step.value = step.value === 4 ? 0 : step.value + 1
}

function advanceFigure() {
  figureStep.value = figureStep.value === figureSteps.length - 1 ? 0 : figureStep.value + 1
}
</script>

<template>
  <section class="cboc-deep-dive">
    <header class="cboc-intro">
      <div><span class="eyebrow">PUNTOS 2 Y 3 DEL PDF · CBoC</span><h2>¿Para qué sirve CBoC?</h2></div>
      <p><strong>CBoC —Optimización de Control Basado en Costos—</strong> se presenta en el material como una plataforma que hace viable procesar Big Data con escalabilidad horizontal y tolerancia a fallos. Distribuye almacenamiento y trabajo entre servidores comunes, mientras coordina quién tiene autoridad cuando algún componente falla.</p>
    </header>

    <div class="cboc-purpose-grid">
      <div><span>ESCALAR</span><strong>Agregar nodos</strong><p>Aumenta almacenamiento y cómputo sin depender de una sola máquina enorme.</p></div>
      <div><span>RESISTIR</span><strong>Continuar ante fallos</strong><p>Réplicas y procesos sustitutos evitan que una caída individual detenga todo.</p></div>
      <div><span>COORDINAR</span><strong>Una autoridad válida</strong><p>Bloqueos y consenso controlan quién dirige o modifica recursos compartidos.</p></div>
    </div>

    <section class="cboc-layers">
      <div class="section-heading"><div><span class="eyebrow">TRES SUBSISTEMAS · CAPAS DE RESPONSABILIDAD</span><h2>No son tres copias de lo mismo.</h2></div></div>
      <div class="layer-diagram">
        <div class="layer-selector">
          <button v-for="(layer, key) in layers" :key="key" :class="{ active: activeLayer === key }" @click="activeLayer = key as Layer">
            <span>{{ layer.number }}</span><b>{{ layer.icon }}</b><strong>{{ layer.name }}</strong><small>{{ layer.role }}</small>
          </button>
        </div>
        <Transition name="layer-detail" mode="out-in">
          <article :key="activeLayer" class="layer-detail">
            <span class="layer-badge">{{ currentLayer.number }} · {{ currentLayer.role }}</span><h3>{{ currentLayer.name }}</h3>
            <div><span>QUÉ HACE</span><p>{{ currentLayer.purpose }}</p></div>
            <div><span>EN QUÉ AYUDA</span><p>{{ currentLayer.helps }}</p></div>
            <div class="layer-example"><span>EJEMPLO</span><p>{{ currentLayer.example }}</p></div>
            <div class="layer-warning"><span>NO CONFUNDIR</span><p>{{ currentLayer.misconception }}</p></div>
          </article>
        </Transition>
      </div>
      <p class="layer-clarification"><strong>Cómo nombrarlos en el examen:</strong> el PDF los denomina “tres subsistemas de procesamiento distribuido”. Puedes explicarlos como tres niveles funcionales, pero no como una jerarquía donde uno reemplaza al anterior.</p>
    </section>

    <section class="cboc-failure-lab">
      <div class="section-heading"><div><span class="eyebrow">FIGURA 3 · DIAGRAMA ANIMADO</span><h2>Observa cómo colaboran durante un fallo.</h2></div></div>
      <div class="figure3-original">
        <figure>
          <img src="/tema11-figura-3.png" alt="Figura 3 original: failover de maestros y recuperación de trabajadores coordinados por el bloqueo distribuido">
          <figcaption>Figura 3 original · Tema 11, diapositiva 14. La parte superior izquierda representa maestros; la inferior, el servicio de bloqueo; y la derecha, los trabajadores.</figcaption>
        </figure>
        <div class="figure3-reading">
          <span class="eyebrow">CÓMO LEERLA</span>
          <h3>Dos rutas de recuperación comparten una autoridad.</h3>
          <p><strong>Ruta del trabajador:</strong> (a) supervisar → (b) notificar al maestro → (c) ordenar recuperación.</p>
          <p><strong>Ruta del maestro:</strong> detectar la pérdida del activo → liberar o expirar su bloqueo → promover al standby cuando adquiere el bloqueo exclusivo.</p>
        </div>
      </div>
      <div class="figure3-animation">
        <div class="figure3-scene">
          <div class="scene-masters" :class="{ active: figureStep === 0 || figureStep >= 5 }">
            <span>MAESTROS DE ARCHIVOS / TABLAS</span>
            <div><b :class="{ failed: figureStep === 5 }">ACTIVO</b><i>⇄</i><b :class="{ promoted: figureStep >= 5 }">STANDBY</b></div>
            <small>{{ figureStep < 5 ? 'Activo posee el bloqueo' : figureStep === 5 ? 'Standby adquiere autoridad' : 'Nuevo maestro activo' }}</small>
          </div>
          <div class="scene-lock" :class="{ active: figureStep === 0 || figureStep === 1 || figureStep === 3 || figureStep >= 5 }">
            <span>CONSENSO + BLOQUEO EXCLUSIVO</span><strong>⛓</strong><small>Coordina autoridad y supervisión</small>
          </div>
          <div class="scene-workers" :class="{ active: figureStep >= 1 && figureStep <= 4 || figureStep === 6 }">
            <span>PROCESOS TRABAJADORES</span>
            <div><b :class="{ failed: figureStep >= 2 && figureStep < 5 }">W1</b><b :class="{ recovering: figureStep === 4 }">W2</b><b>W3</b></div>
            <small>{{ figureStep < 2 ? 'Procesan y replican datos' : figureStep < 4 ? 'W1 no responde' : 'W2 recupera la carga de W1' }}</small>
          </div>
          <span class="scene-arrow arrow-a" :class="{ active: figureStep === 1 }">(a) monitoreo</span>
          <span class="scene-arrow arrow-b" :class="{ active: figureStep === 3 }">(b) notificación</span>
          <span class="scene-arrow arrow-c" :class="{ active: figureStep === 4 }">(c) instrucción</span>
        </div>
        <div class="figure3-step-panel">
          <span>PASO {{ figureStep + 1 }} / {{ figureSteps.length }}</span>
          <h3>{{ figureSteps[figureStep]!.label }}</h3>
          <p>{{ figureSteps[figureStep]!.detail }}</p>
          <div class="figure3-step-dots"><button v-for="(_, index) in figureSteps" :key="index" :class="{ active: figureStep === index }" :aria-label="'Ver paso ' + (index + 1)" @click="figureStep = index"></button></div>
          <button class="primary" @click="advanceFigure">{{ figureStep === figureSteps.length - 1 ? 'Reiniciar animación ↻' : 'Mostrar siguiente paso →' }}</button>
        </div>
      </div>
      <div class="figure3-guide">
        <article><span>ARRIBA IZQUIERDA</span><strong>Maestros de archivos y tablas</strong><p>Hay un maestro activo y otro en espera. El activo posee el bloqueo; el standby espera adquirirlo. Si el activo falla, se libera o expira su autoridad y el standby obtiene el bloqueo para ejecutar el <em>failover</em>.</p></article>
        <article><span>PARTE INFERIOR</span><strong>Servidores de bloqueo distribuido</strong><p>Replican el estado del bloqueo mediante un protocolo de consenso. Su maestro concede un único bloqueo exclusivo y evita que dos procesos se consideren activos simultáneamente.</p></article>
        <article><span>LADO DERECHO</span><strong>Trabajadores de archivos y tablas</strong><p>Procesan solicitudes y administran partes redundantes de los datos. Si uno falla, otro trabajador puede recuperar sus datos y retomar el procesamiento.</p></article>
      </div>
      <div class="figure3-arrows">
        <div><b>(a)</b><span>Alive monitoring</span><p>El subsistema de bloqueo comprueba periódicamente si los trabajadores siguen activos.</p></div>
        <div><b>(b)</b><span>Failure notification</span><p>Al detectar un trabajador caído, informa al maestro activo de archivos o tablas.</p></div>
        <div><b>(c)</b><span>Recovery instruction</span><p>El maestro ordena a un trabajador disponible recuperar los datos y continuar la tarea.</p></div>
      </div>
      <p class="figure3-conclusion"><strong>Lectura completa:</strong> el bloqueo distribuido no recupera directamente los datos. Detecta y coordina; el maestro decide la reasignación; los trabajadores y las réplicas ejecutan la recuperación. Si quien falla es el propio maestro, el bloqueo exclusivo permite promover al maestro en espera.</p>
      <div class="failure-tabs">
        <button :class="{ active: failure === 'master' }" @click="runFailure('master')">Falla del maestro</button>
        <button :class="{ active: failure === 'worker' }" @click="runFailure('worker')">Falla de trabajador</button>
        <button :class="{ active: failure === 'network' }" @click="runFailure('network')">Falla de red</button>
      </div>
      <div class="failure-board">
        <div class="failure-flow">
          <div v-for="(stage, index) in currentFailure.stages" :key="stage[1]" :class="{ active: step >= index + 1 }">
            <span>{{ stage[0] }}</span><strong>{{ stage[1] }}</strong><small>{{ stage[2] }}</small><i v-if="index < 3">→</i>
          </div>
        </div>
        <div class="failure-controls">
          <p><strong>{{ currentFailure.title }}.</strong> {{ step === 4 ? currentFailure.result : 'Avanza para seguir la secuencia de recuperación.' }}</p>
          <button class="primary" @click="advance">{{ step === 4 ? 'Reiniciar animación ↻' : 'Siguiente evento →' }}</button>
        </div>
      </div>
    </section>

    <div class="cboc-exam-answer">
      <span class="eyebrow">RESPUESTA CORTA PARA EL EXAMEN</span>
      <p>CBoC sirve para procesar grandes volúmenes de datos de forma escalable y tolerante a fallos. El subsistema de archivos <strong>almacena y replica</strong>, el de tablas <strong>organiza y distribuye el acceso</strong>, y el de bloqueo <strong>coordina autoridad, supervisión y recuperación</strong>. Juntos permiten continuar cuando falla un maestro, un trabajador o una rama de red.</p>
    </div>
  </section>
</template>
