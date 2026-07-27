<script setup lang="ts">
interface Flashcard { level: string; q: string; a: string }
interface QuizQuestion { area: string; q: string; options: string[]; correct: number; explanation: string }

defineProps<{
  flashcards: Flashcard[]; flashIndex: number; flashRevealed: boolean; streak: number
  quizQuestions: QuizQuestion[]; quizStarted: boolean; quizFinished: boolean
  quizIndex: number; quizAnswers: (number | null)[]; quizScore: number
}>()
defineEmits<{
  toggleFlash: []; nextFlash: [knew: boolean]; startQuiz: []; answerQuiz: [option: number]
  selectQuiz: [index: number]; finishQuiz: []
}>()
</script>

<template>
  <section class="page">
    <header class="page-header"><span class="eyebrow">QUIZ TEMA 11 · ACTIVE RECALL</span><h1>Practica el razonamiento,<br>no el reconocimiento.</h1><p>Flashcards y simulacro avanzado de los conceptos del Tema 11.</p></header>
    <div class="practice-layout">
      <div>
        <div class="practice-meta"><span>FLASHCARD {{ flashIndex + 1 }} / {{ flashcards.length }}</span><span>RACHA: {{ streak }}</span><span>{{ flashcards[flashIndex]!.level }}</span></div>
        <button class="flashcard" :class="{ revealed: flashRevealed }" @click="$emit('toggleFlash')">
          <div v-if="!flashRevealed"><span class="eyebrow">PREGUNTA TÉCNICA</span><h2>{{ flashcards[flashIndex]!.q }}</h2><small>TOCA PARA REVELAR ↻</small></div>
          <div v-else><span class="eyebrow">RESPUESTA MODELO</span><p>{{ flashcards[flashIndex]!.a }}</p><small>TOCA PARA VOLVER ↻</small></div>
        </button>
        <div v-if="flashRevealed" class="recall-actions"><button @click="$emit('nextFlash', true)">✓ Lo sabía</button><button @click="$emit('nextFlash', false)">↻ Necesito repasar</button></div>
      </div>
      <aside class="practice-guide"><span class="eyebrow">MÉTODO</span><h3>Cómo usar estas tarjetas</h3><ol><li>Formula una respuesta completa.</li><li>Incluye mecanismo y trade-off.</li><li>Revela y detecta la omisión.</li><li>Marca con honestidad.</li></ol></aside>
    </div>
    <section class="quiz-section">
      <div v-if="!quizStarted" class="quiz-intro">
        <div><span class="eyebrow">SIMULACRO DE ALTA DIFICULTAD</span><h2>{{ quizQuestions.length }} preguntas para detectar si realmente entendiste.</h2><p>Las alternativas incorrectas son técnicamente plausibles. Cada respuesta se califica inmediatamente y muestra el fundamento.</p><ul><li>{{ quizQuestions.length }} casos de análisis</li><li>Feedback inmediato fundamentado</li><li>Revisión completa al finalizar</li></ul></div>
        <button class="primary quiz-start" @click="$emit('startQuiz')">Iniciar simulacro <span>→</span></button>
      </div>
      <div v-else-if="!quizFinished" class="quiz-runner">
        <div class="quiz-top"><div><span class="eyebrow">SIMULACRO · PREGUNTA {{ quizIndex + 1 }} DE {{ quizQuestions.length }}</span><h2>{{ quizQuestions[quizIndex]!.area }}</h2></div><strong>{{ Math.round(((quizIndex + 1) / quizQuestions.length) * 100) }}%</strong></div>
        <div class="quiz-progress"><span :style="{ width: ((quizIndex + 1) / quizQuestions.length) * 100 + '%' }"></span></div>
        <article class="quiz-card">
          <span class="difficulty">DIFICULTAD ALTA · ANÁLISIS</span><h3>{{ quizQuestions[quizIndex]!.q }}</h3>
          <div class="quiz-options"><button v-for="(option, oi) in quizQuestions[quizIndex]!.options" :key="option" :class="{ selected: quizAnswers[quizIndex] === oi }" @click="$emit('answerQuiz', oi)"><span>{{ String.fromCharCode(65 + oi) }}</span><strong>{{ option }}</strong></button></div>
          <Transition name="instant-feedback">
            <div v-if="quizAnswers[quizIndex] !== null" class="instant-feedback" :class="quizAnswers[quizIndex] === quizQuestions[quizIndex]!.correct ? 'is-correct' : 'is-wrong'">
              <strong>{{ quizAnswers[quizIndex] === quizQuestions[quizIndex]!.correct ? '✓ Respuesta correcta' : '× Respuesta incorrecta' }}</strong>
              <p v-if="quizAnswers[quizIndex] !== quizQuestions[quizIndex]!.correct"><span>La respuesta correcta es:</span> {{ quizQuestions[quizIndex]!.options[quizQuestions[quizIndex]!.correct] }}</p>
              <p><span>Fundamento:</span> {{ quizQuestions[quizIndex]!.explanation }}</p>
            </div>
          </Transition>
        </article>
        <div class="quiz-nav">
          <button :disabled="quizIndex === 0" @click="$emit('selectQuiz', quizIndex - 1)">← Anterior</button>
          <div class="quiz-dots"><button v-for="(_, qi) in quizQuestions" :key="qi" :class="{ current: quizIndex === qi, answered: quizAnswers[qi] !== null }" :aria-label="'Ir a pregunta ' + (qi + 1)" @click="$emit('selectQuiz', qi)"></button></div>
          <button v-if="quizIndex < quizQuestions.length - 1" :disabled="quizAnswers[quizIndex] === null" @click="$emit('selectQuiz', quizIndex + 1)">Siguiente →</button>
          <button v-else class="finish" :disabled="quizAnswers.some(answer => answer === null)" @click="$emit('finishQuiz')">Finalizar</button>
        </div>
        <p v-if="quizAnswers.some(answer => answer === null) && quizIndex === quizQuestions.length - 1" class="quiz-warning">Debes responder todas las preguntas antes de finalizar.</p>
      </div>
      <div v-else class="quiz-results">
        <header>
          <span class="eyebrow">RESULTADO · QUIZ TEMA 11</span><div class="score-ring"><strong>{{ quizScore }}</strong><span>/ {{ quizQuestions.length }}</span></div>
          <h2>{{ quizScore / quizQuestions.length >= .85 ? 'Dominio sólido' : quizScore / quizQuestions.length >= .65 ? 'Buen razonamiento, con fisuras' : quizScore / quizQuestions.length >= .5 ? 'Comprensión todavía inestable' : 'Necesitas reconstruir los fundamentos' }}</h2>
          <p>{{ Math.round((quizScore / quizQuestions.length) * 100) }}% de precisión. Revisa especialmente las preguntas falladas y explica por qué cada distractor no es defendible.</p>
          <button class="outline-button" @click="$emit('startQuiz')">Reintentar simulacro ↻</button>
        </header>
        <div class="result-list">
          <details v-for="(question, i) in quizQuestions" :key="question.q" :class="{ correct: quizAnswers[i] === question.correct, wrong: quizAnswers[i] !== question.correct }">
            <summary><span>{{ quizAnswers[i] === question.correct ? '✓' : '×' }}</span><strong>{{ i + 1 }}. {{ question.q }}</strong><b>+</b></summary>
            <div><p><span>TU RESPUESTA</span>{{ question.options[quizAnswers[i] ?? 0] }}</p><p><span>RESPUESTA CORRECTA</span>{{ question.options[question.correct] }}</p><p><span>RAZONAMIENTO</span>{{ question.explanation }}</p></div>
          </details>
        </div>
      </div>
    </section>
  </section>
</template>
