<script setup lang="ts">
import { computed, ref } from 'vue'
const questions = [
  { q: '¿Qué diferencia con mayor precisión sintaxis de semántica?', o: ['Sintaxis divide caracteres; semántica corrige ortografía.', 'Sintaxis representa relaciones gramaticales; semántica interpreta su significado.', 'Son la misma fase.', 'Semántica solo cuenta frecuencias.'], c: 1, e: 'La sintaxis modela estructura y dependencias; la semántica vincula esa estructura con significado.' },
  { q: '“Vi al hombre con el telescopio” muestra principalmente:', o: ['Ambigüedad sintáctica sobre quién tiene o usa el telescopio.', 'Un error léxico.', 'Que tokenizar produce una interpretación única.', 'Que todo texto requiere traducción.'], c: 0, e: 'El sintagma puede modificar “vi” o “hombre”; contexto adicional decide.' },
  { q: '¿Por qué la búsqueda semántica supera a veces las palabras clave?', o: ['Elimina todo índice.', 'Representa intención y cercanía de significado aunque cambie el vocabulario.', 'Siempre devuelve más.', 'No evalúa relevancia.'], c: 1, e: 'Conecta paráfrasis y conceptos relacionados, aunque todavía necesita indexación y métricas.' },
  { q: '¿Qué describe mejor el enfoque por reglas?', o: ['Aprende sin intervención.', 'Es explicable y determinista, pero mantener cobertura y excepciones cuesta.', 'Solo usa redes.', 'Solo asigna probabilidades.'], c: 1, e: 'Es fuerte en dominios estables, pero escala mal ante la variedad lingüística.' },
  { q: '¿Qué aportó el PLN estadístico?', o: ['Eliminó errores.', 'Estimó interpretaciones a partir de datos y probabilidades.', 'Creó conciencia.', 'Prohibió vectores.'], c: 1, e: 'n-gramas, HMM o regresión aprenden patrones y cuantifican incertidumbre.' },
  { q: 'En “No estuvo mal”, marcar negativo por “mal” falla por:', o: ['No modelar alcance de negación y composición.', 'Usar muchos idiomas.', 'Aplicar sintaxis.', 'Reconocer contexto.'], c: 0, e: 'El significado no es la suma de palabras aisladas.' },
  { q: '¿Qué corrección requiere el flujo que termina en entrenamiento?', o: ['Se entrena en toda consulta.', 'Entrenamiento e inferencia son iguales.', 'El entrenamiento suele ser offline; producción normalmente ejecuta inferencia.', 'Extracción ocurre al final.'], c: 2, e: 'Conviene separar desarrollo del modelo y ejecución del modelo.' },
  { q: '¿Por qué la fluidez de un LLM no garantiza verdad?', o: ['Optimiza secuencias plausibles y puede afirmar sin respaldo.', 'Nunca escribe bien.', 'Solo usa IF-THEN.', 'No usa vectores.'], c: 0, e: 'Verosimilitud lingüística no equivale a verificación factual.' },
  { q: 'En salud, evaluar solo exactitud promedio omite:', o: ['Costos asimétricos, calibración, sesgo y apelación.', 'El tamaño de letra.', 'Que la IA es infalible.', 'Que no existe contexto.'], c: 0, e: 'El impacto depende del tipo de error y del grupo afectado.' },
  { q: '¿Qué arquitectura es defendible en un dominio legal?', o: ['LLM sin controles.', 'Modelo + fuentes recuperadas + reglas duras + revisión humana.', 'Solo palabras clave.', 'Generación sin registro.'], c: 1, e: 'El enfoque híbrido combina cobertura, trazabilidad y restricciones.' },
  { q: 'BoW/TF-IDF frente a embeddings modernos:', o: ['Siempre conservan contexto completo.', 'Son conteos dispersos; embeddings son vectores densos y pueden ser contextuales.', 'No representan palabras.', 'Son Transformers.'], c: 1, e: 'Los primeros capturan importancia léxica; los embeddings codifican regularidades semánticas.' },
  { q: '¿El PLN realmente “entiende”?', o: ['Sí, porque responde rápido.', 'No admite debate.', 'Depende del criterio: competencia funcional no prueba conciencia ni comprensión humana.', 'Sí, porque usa datos.'], c: 2, e: 'Hay que separar desempeño observable de afirmaciones sobre conciencia.' },
]
const index = ref(0)
const answers = ref<(number | null)[]>(Array(questions.length).fill(null))
const finished = ref(false)
const score = computed(() => answers.value.reduce<number>((s, a, i) => s + (a === questions[i].c ? 1 : 0), 0))
function reset(){ answers.value = Array(questions.length).fill(null); index.value = 0; finished.value = false }
</script>
<template>
  <section class="page topic14-quiz">
    <header class="page-header"><span class="eyebrow">EVALUACIÓN · DIFICULTAD ALTA</span><h1>Quiz de <em>PLN.</em></h1><p>Distingue conceptos, aplica el pipeline y justifica límites.</p></header>
    <div v-if="!finished" class="t14-quiz-card">
      <div class="quiz-top"><div><span class="eyebrow">PREGUNTA {{ index + 1 }} DE {{ questions.length }}</span><h2>Razonamiento aplicado</h2></div><strong>{{ Math.round((index + 1) / questions.length * 100) }}%</strong></div>
      <div class="quiz-progress"><span :style="{ width: ((index + 1) / questions.length * 100) + '%' }"></span></div>
      <h3>{{ questions[index].q }}</h3>
      <div class="quiz-options"><button v-for="(option, i) in questions[index].o" :key="option" :class="{ selected: answers[index] === i }" @click="answers[index] = i"><span>{{ String.fromCharCode(65 + i) }}</span><strong>{{ option }}</strong></button></div>
      <Transition name="instant-feedback"><div v-if="answers[index] !== null" class="instant-feedback" :class="answers[index] === questions[index].c ? 'is-correct' : 'is-wrong'"><strong>{{ answers[index] === questions[index].c ? '✓ Correcto' : '✕ Revisa el razonamiento' }}</strong><p>{{ questions[index].e }}</p></div></Transition>
      <div class="quiz-nav"><button :disabled="index === 0" @click="index--">← Anterior</button><div></div><button v-if="index < questions.length - 1" :disabled="answers[index] === null" @click="index++">Siguiente →</button><button v-else class="finish" :disabled="answers.some(a => a === null)" @click="finished = true">Finalizar</button></div>
    </div>
    <div v-else class="topic14-results"><span class="eyebrow">RESULTADO</span><h1>{{ score }} / {{ questions.length }}</h1><p>{{ score >= 10 ? 'Dominio sólido para defender matices.' : score >= 8 ? 'Buen nivel; revisa tus errores.' : 'Regresa a los diagramas y repite el laboratorio.' }}</p><button class="outline-button" @click="reset">Reintentar</button><details v-for="(q, i) in questions" :key="q.q" :class="answers[i] === q.c ? 'correct' : 'wrong'"><summary><span>{{ answers[i] === q.c ? '✓' : '✕' }}</span><strong>{{ q.q }}</strong></summary><p>{{ q.e }}</p></details></div>
  </section>
</template>
