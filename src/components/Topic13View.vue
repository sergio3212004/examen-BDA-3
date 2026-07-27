<script setup lang="ts">
import { computed, ref } from 'vue'

const architectureStep = ref(0)
const selectedType = ref<'contenido'|'colaborativo'|'conocimiento'|'hibrido'>('contenido')
const challenge = ref(0)
const streamStep = ref(0)

const types = {
  contenido:{name:'Basado en contenido',signal:'Perfil del ítem + historial del mismo usuario',logic:'Busca similitud entre atributos: “consumiste ML; te sugiero Deep Learning”.',advantage:'No depende de otros usuarios y personaliza desde el historial propio.',risk:'Sobre-especialización: recomienda más de lo mismo y reduce descubrimiento.'},
  colaborativo:{name:'Filtrado colaborativo',signal:'Interacciones de usuarios parecidos',logic:'Usuarios A y B coinciden; lo que A consumió y B no, se propone a B.',advantage:'Descubre relaciones sin describir manualmente los productos.',risk:'Cold start y dispersión; nuevos usuarios o ítems carecen de señales.'},
  conocimiento:{name:'Basado en conocimiento',signal:'Restricciones + reglas expertas',logic:'Presupuesto, ciudad, temporada y personas filtran hoteles compatibles.',advantage:'Útil en compras poco frecuentes, costosas o con restricciones duras.',risk:'Adquirir y mantener conocimiento experto puede ser caro.'},
  hibrido:{name:'Sistema híbrido',signal:'Contenido + colaboración + contexto + modelos',logic:'Combina puntuaciones o encadena modelos para cubrir debilidades mutuas.',advantage:'Mejor cobertura, robustez y respuesta al cold start.',risk:'Más complejidad, latencia, monitoreo y dificultad para atribuir resultados.'},
}
const currentType = computed(()=>types[selectedType.value])
const challenges = [
  ['Cold start','Faltan interacciones para usuarios o productos nuevos.','Onboarding, popularidad contextual, atributos de contenido y exploración controlada.'],
  ['Escalabilidad','Millones de usuarios × productos hacen inviable puntuar todo.','Candidatos aproximados, partición, caché y ranking en dos etapas.'],
  ['Dispersión','Cada usuario toca una fracción mínima del catálogo.','Factorización, embeddings, señales implícitas y regularización.'],
  ['Calidad','Cuentas compartidas, ruido o eventos duplicados deforman preferencias.','Validación, identidad, deduplicación, linaje y ponderación de señales.'],
  ['Concept drift','Los intereses cambian por tiempo, estación y contexto.','Ventanas temporales, decaimiento, reentrenamiento y detección de deriva.'],
  ['Sesgo','Popularidad y exposición crean bucles de retroalimentación.','Re-ranking, cuotas de exploración y métricas por proveedor/grupo.'],
  ['Privacidad','Historial, ubicación y compras revelan información sensible.','Minimización, consentimiento, retención limitada y controles de acceso.'],
  ['Tiempo real','La siguiente recomendación debe incorporar el evento recién ocurrido.','Kafka/Flink, estado en streaming, feature store y serving de baja latencia.'],
  ['Interpretabilidad','El usuario o auditor necesita saber por qué se recomendó.','Razones fieles: similitud, regla, historial o atributos decisivos.'],
  ['Precisión vs diversidad','Optimizar clic inmediato puede estrechar el catálogo.','Objetivo multi-métrica: relevancia, novedad, diversidad y serendipia.'],
]
</script>

<template>
  <section class="page topic13-page">
    <header class="topic13-hero">
      <div><span class="eyebrow">BASE DE DATOS AVANZADA · TEMA 13</span><h1>Sistemas de <em>Recomendación.</em></h1><p>Transforman interacciones masivas en una lista personalizada de elementos relevantes. En Big Data, el problema no es solo predecir gusto: también es capturar eventos, entrenar, servir resultados en milisegundos y evitar efectos adversos.</p></div>
      <aside><span class="eyebrow">MODELO MENTAL</span><strong>Señales → candidatos → puntuación → reordenamiento → recomendación → nueva interacción.</strong><a class="outline-button" href="/[13-1]%20BDA%20-%20Clase.pdf" target="_blank">Abrir PDF original ↗</a></aside>
    </header>

    <section class="t13-value">
      <div><span class="eyebrow">¿POR QUÉ IMPORTAN?</span><h2>El catálogo crece más rápido que la atención.</h2></div>
      <p>Con millones de usuarios, ítems y eventos, explorar manualmente es imposible. La recomendación reduce sobrecarga de elección y puede aumentar ventas, fidelización, permanencia e ingresos. Esos beneficios solo son válidos si se comparan contra una línea base y no deterioran confianza, diversidad o privacidad.</p>
      <div class="t13-benefits"><span>Ventas</span><span>Fidelización</span><span>Personalización</span><span>Permanencia</span><span>Marketing</span><span>Experiencia</span></div>
    </section>

    <section class="t14-section">
      <span class="eyebrow">ARQUITECTURA DEL DOCUMENTO</span><h2>Cuatro componentes y un ciclo de retroalimentación.</h2>
      <div class="t13-architecture">
        <button v-for="(item,i) in [['Usuarios','Compras, clics, búsquedas, calificaciones, tiempo y comentarios'],['Productos','Películas, música, libros, noticias, cursos, hoteles o productos'],['Datos','Perfil, contexto, historial, frecuencia y categorías favoritas'],['Algoritmos','Analizan señales, generan candidatos, puntúan y ordenan']]" :key="item[0]" :class="{active:i<=architectureStep}" @click="architectureStep=i"><span>0{{i+1}}</span><strong>{{item[0]}}</strong><small>{{item[1]}}</small><i v-if="i<3">→</i></button>
      </div>
      <button class="primary t13-advance" @click="architectureStep=(architectureStep+1)%4">{{architectureStep===3?'Reiniciar ciclo ↻':'Avanzar arquitectura →'}}</button>
      <div class="t14-exam-note"><strong>Trampa frecuente:</strong> el algoritmo no “crea” preferencias; infiere una utilidad a partir de señales observadas. Clic, compra y tiempo de permanencia expresan intenciones distintas y deben ponderarse.</div>
    </section>

    <section class="t14-section">
      <span class="eyebrow">TIPOS DE SISTEMAS</span><h2>Misma meta, distinta fuente de evidencia.</h2>
      <div class="t14-tabs t13-tabs"><button v-for="(item,key) in types" :key="key" :class="{active:selectedType===key}" @click="selectedType=key as keyof typeof types">{{item.name}}</button></div>
      <Transition name="t14-shift" mode="out-in"><article :key="selectedType" class="t13-type-card"><header><span>{{currentType.name}}</span><h3>{{currentType.signal}}</h3></header><div><span>LÓGICA</span><p>{{currentType.logic}}</p></div><div class="good"><span>VENTAJA</span><p>{{currentType.advantage}}</p></div><div class="risk"><span>LÍMITE</span><p>{{currentType.risk}}</p></div></article></Transition>
    </section>

    <section class="t14-section">
      <span class="eyebrow">TÉCNICAS DE MACHINE LEARNING</span><h2>Qué aprende cada familia.</h2>
      <div class="t13-ml-grid">
        <article><span>SUPERVISADO</span><h3>Predicción explícita</h3><p>Clasificación o regresión estima compra, clic o rating usando ejemplos etiquetados.</p></article>
        <article><span>NO SUPERVISADO</span><h3>Segmentación</h3><p>Clustering agrupa usuarios frecuentes, ocasionales o de patrones similares.</p></article>
        <article><span>FACTORIZACIÓN</span><h3>Factores latentes</h3><p>Aproxima la matriz usuario–ítem con vectores compactos que capturan gustos ocultos.</p></article>
        <article><span>DEEP LEARNING</span><h3>Relaciones complejas</h3><p>Embeddings y redes combinan secuencia, contenido, contexto y múltiples señales.</p></article>
      </div>
      <div class="t13-matrix-demo"><div><span>USUARIO × ÍTEM</span><div class="mini-matrix"><i>5</i><i></i><i>4</i><i></i><i></i><i>5</i><i></i><i>2</i><i></i></div></div><b>≈</b><div><span>VECTOR USUARIO</span><code>[acción .82, educativo .61]</code></div><b>×</b><div><span>VECTOR ÍTEM</span><code>[acción .77, educativo .54]</code></div></div>
    </section>

    <section class="t14-section">
      <span class="eyebrow">BIG DATA FRAMEWORKS</span><h2>Almacenar, calcular, transportar y reaccionar.</h2>
      <div class="t13-stack"><article><strong>Hadoop</strong><span>Data lake / batch</span><p>Almacenamiento distribuido y procesamiento histórico masivo.</p></article><i>→</i><article><strong>Spark</strong><span>Features / entrenamiento</span><p>Procesa en memoria, prepara datos y entrena modelos distribuidos.</p></article><i>→</i><article><strong>Kafka</strong><span>Log de eventos</span><p>Captura clics, vistas, compras y ubicación en tiempo real.</p></article><i>→</i><article><strong>Flink</strong><span>Streaming con estado</span><p>Actualiza señales y recomendaciones conforme llegan eventos.</p></article></div>
      <div class="t13-stream"><button v-for="(name,i) in ['Clic del usuario','Evento en Kafka','Estado en Flink','Ranking actualizado']" :key="name" :class="{active:i<=streamStep}" @click="streamStep=i"><span>0{{i+1}}</span><strong>{{name}}</strong></button></div><button class="text-button" @click="streamStep=(streamStep+1)%4">Simular siguiente evento →</button>
    </section>

    <section class="t14-section">
      <span class="eyebrow">LOS DIEZ DESAFÍOS DEL PDF</span><h2>La precisión es solo una parte del sistema.</h2>
      <div class="t13-challenge-layout"><nav><button v-for="(item,i) in challenges" :key="item[0]" :class="{active:challenge===i}" @click="challenge=i"><span>{{String(i+1).padStart(2,'0')}}</span>{{item[0]}}</button></nav><Transition name="t14-shift" mode="out-in"><article :key="challenge"><span>DESAFÍO {{String(challenge+1).padStart(2,'0')}}</span><h3>{{challenges[challenge][0]}}</h3><div><small>PROBLEMA</small><p>{{challenges[challenge][1]}}</p></div><div class="solution"><small>RESPUESTA ARQUITECTÓNICA</small><p>{{challenges[challenge][2]}}</p></div></article></Transition></div>
    </section>

    <section class="t14-section">
      <span class="eyebrow">MÉTRICAS PARA UN EXAMEN DIFÍCIL</span><h2>No confundas predicción offline con valor real.</h2>
      <div class="t13-metrics"><article><span>OFFLINE</span><strong>Precision@K · Recall@K · NDCG</strong><p>Comparan el ranking con interacciones conocidas, pero heredan sesgo de exposición.</p></article><article><span>ONLINE</span><strong>CTR · conversión · retención</strong><p>Un A/B test estima efecto causal sobre conducta, vigilando métricas de protección.</p></article><article><span>CALIDAD AMPLIA</span><strong>Diversidad · novedad · cobertura</strong><p>Evitan que “precisión” signifique repetir únicamente los ítems populares.</p></article></div>
      <blockquote>Un recomendador responsable optimiza utilidad a largo plazo bajo restricciones de privacidad, equidad, diversidad, latencia y explicabilidad.</blockquote>
    </section>
  </section>
</template>
