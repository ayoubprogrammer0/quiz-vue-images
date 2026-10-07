<script setup>
import { computed, ref } from 'vue'

const questions = [
  {
    image: '/images/tour-eiffel.svg',
    alt: 'Illustration de la tour Eiffel au coucher du soleil',
    category: 'Architecture',
    prompt: 'Dans quelle ville se trouve ce monument ?',
    choices: ['Paris', 'Rome', 'Lisbonne'],
    answer: 'Paris',
  },
  {
    image: '/images/abeille.svg',
    alt: 'Illustration d’une abeille près de fleurs',
    category: 'Nature',
    prompt: 'Quel rôle joue principalement cet insecte ?',
    choices: ['Polliniser les fleurs', 'Produire du lait', 'Chasser les moustiques'],
    answer: 'Polliniser les fleurs',
  },
  {
    image: '/images/mars.svg',
    alt: 'Illustration de la planète Mars dans l’espace',
    category: 'Espace',
    prompt: 'Quelle planète est représentée ?',
    choices: ['Mars', 'Jupiter', 'Mercure'],
    answer: 'Mars',
  },
  {
    image: '/images/portrait.svg',
    alt: 'Illustration d’un portrait de femme dans un paysage',
    category: 'Art',
    prompt: 'Qui a peint ce célèbre portrait ?',
    choices: ['Léonard de Vinci', 'Claude Monet', 'Frida Kahlo'],
    answer: 'Léonard de Vinci',
  },
  {
    image: '/images/baleine.svg',
    alt: 'Illustration d’une grande baleine bleue sous la mer',
    category: 'Océans',
    prompt: 'Quel est cet animal marin ?',
    choices: ['Baleine bleue', 'Requin-baleine', 'Dauphin'],
    answer: 'Baleine bleue',
  },
]

const questionIndex = ref(0)
const score = ref(0)
const selectedAnswer = ref(null)
const isFinished = ref(false)

const currentQuestion = computed(() => questions[questionIndex.value])
const progress = computed(() => ((questionIndex.value + 1) / questions.length) * 100)
const isCorrect = computed(() => selectedAnswer.value === currentQuestion.value.answer)

function chooseAnswer(choice) {
  if (selectedAnswer.value !== null) return

  selectedAnswer.value = choice
  if (choice === currentQuestion.value.answer) score.value += 1
}

function nextQuestion() {
  if (questionIndex.value === questions.length - 1) {
    isFinished.value = true
    return
  }

  questionIndex.value += 1
  selectedAnswer.value = null
}

function restartQuiz() {
  questionIndex.value = 0
  score.value = 0
  selectedAnswer.value = null
  isFinished.value = false
}
</script>

<template>
  <div class="app-shell">
    <header class="topbar">
      <a class="wordmark" href="#top" aria-label="Quiz en images, accueil">
        <span class="wordmark-icon" aria-hidden="true"><span></span><span></span><span></span></span>
        <span>Œil <b>curieux</b></span>
      </a>
      <div class="topbar-note"><span class="live-dot"></span> Le quiz du jour</div>
      <div class="topbar-score" aria-live="polite">
        <span class="score-star" aria-hidden="true">✳</span>
        <span>{{ score }} <span class="score-label">point{{ score > 1 ? 's' : '' }}</span></span>
      </div>
    </header>

    <main id="top" class="quiz-main">
      <template v-if="!isFinished">
        <section class="progress-section" aria-label="Progression du quiz">
          <div class="progress-copy">
            <span class="eyebrow">À vous de voir</span>
            <span class="question-count">{{ String(questionIndex + 1).padStart(2, '0') }} <i>/</i> {{ String(questions.length).padStart(2, '0') }}</span>
          </div>
          <div
            class="progress-track"
            role="progressbar"
            :aria-valuenow="questionIndex + 1"
            :aria-valuemin="1"
            :aria-valuemax="questions.length"
            :aria-label="`Question ${questionIndex + 1} sur ${questions.length}`"
          >
            <span :style="{ width: `${progress}%` }"></span>
          </div>
        </section>

        <section class="question-layout" :key="questionIndex">
          <div class="image-panel">
            <div class="image-meta"><span>Image {{ String(questionIndex + 1).padStart(2, '0') }}</span><span>•</span><span>{{ currentQuestion.category }}</span></div>
            <img class="question-image" :src="currentQuestion.image" :alt="currentQuestion.alt" />
            <div class="image-caption"><span class="caption-mark" aria-hidden="true">↗</span> Regarde bien, chaque détail compte.</div>
          </div>

          <div class="answer-panel">
            <p class="question-kicker">Question {{ questionIndex + 1 }}</p>
            <h1>{{ currentQuestion.prompt }}</h1>
            <p class="instruction">Choisissez la bonne réponse.</p>

            <div class="choices" aria-label="Réponses possibles">
              <button
                v-for="(choice, index) in currentQuestion.choices"
                :key="choice"
                class="choice-button"
                :class="{
                  'is-correct': selectedAnswer !== null && choice === currentQuestion.answer,
                  'is-wrong': selectedAnswer === choice && !isCorrect,
                }"
                type="button"
                :disabled="selectedAnswer !== null"
                :aria-pressed="selectedAnswer === choice"
                @click="chooseAnswer(choice)"
              >
                <span class="choice-letter">{{ String.fromCharCode(65 + index) }}</span>
                <span class="choice-text">{{ choice }}</span>
                <span class="choice-mark" aria-hidden="true">
                  {{ selectedAnswer !== null && choice === currentQuestion.answer ? '✓' : selectedAnswer === choice ? '×' : '↗' }}
                </span>
              </button>
            </div>

            <div v-if="selectedAnswer !== null" class="answer-feedback" :class="{ 'feedback-wrong': !isCorrect }" role="status">
              <span class="feedback-icon" aria-hidden="true">{{ isCorrect ? '✓' : '!' }}</span>
              <span>{{ isCorrect ? 'Bien vu !' : `Pas tout à fait. La réponse était ${currentQuestion.answer}.` }}</span>
            </div>

            <button v-if="selectedAnswer !== null" class="next-button" type="button" @click="nextQuestion">
              {{ questionIndex === questions.length - 1 ? 'Voir mon résultat' : 'Question suivante' }}
              <span aria-hidden="true">→</span>
            </button>
          </div>
        </section>
      </template>

      <section v-else class="result-screen" aria-live="polite">
        <div class="result-orbit" aria-hidden="true"><span>✳</span></div>
        <p class="eyebrow">Fin de la partie</p>
        <h1>{{ score === questions.length ? 'Un regard infaillible.' : 'Bien joué, explorateur.' }}</h1>
        <p class="result-copy">Vous avez obtenu <strong>{{ score }} point{{ score > 1 ? 's' : '' }}</strong> sur {{ questions.length }}.</p>
        <div class="result-score"><span>{{ String(score).padStart(2, '0') }}</span><i>/</i><span>{{ String(questions.length).padStart(2, '0') }}</span></div>
        <button class="next-button restart-button" type="button" @click="restartQuiz">
          Rejouer le quiz <span aria-hidden="true">↻</span>
        </button>
      </section>
    </main>

    <footer class="footer"><span>Culture générale, en un coup d’œil.</span><span>5 questions <i>•</i> 1 point par bonne réponse</span></footer>
  </div>
</template>

<style>
:root {
  font-family: 'Trebuchet MS', sans-serif;
  color: #1e302b;
  background: #f3f4ee;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-optical-sizing: auto;
  font-weight: 400;
}

* { box-sizing: border-box; }

body {
  min-width: 320px;
  min-height: 100vh;
  margin: 0;
}

button, a { font: inherit; }
button { color: inherit; }

.app-shell {
  --ink: #1e302b;
  --muted: #7d8981;
  --line: #dce1d8;
  --leaf: #326b51;
  --leaf-light: #e5eee5;
  --coral: #e97556;
  display: flex;
  min-height: 100vh;
  flex-direction: column;
  padding: 0 7.2vw;
  overflow: hidden;
  background:
    radial-gradient(ellipse at 5% 72%, rgba(216, 229, 211, .52), transparent 31%),
    radial-gradient(ellipse at 94% 11%, rgba(250, 226, 206, .52), transparent 28%),
    #f3f4ee;
}

.topbar {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  align-items: center;
  min-height: 82px;
  border-bottom: 1px solid var(--line);
  color: var(--ink);
}

.wordmark { display: inline-flex; width: fit-content; align-items: center; gap: 10px; color: inherit; text-decoration: none; font-size: 15px; font-weight: 600; }
.wordmark b { color: var(--leaf); font-weight: 700; }
.wordmark-icon { display: grid; width: 24px; height: 24px; grid-template-columns: repeat(3, 1fr); align-items: end; gap: 2px; padding: 4px; border-radius: 7px; background: var(--leaf); }
.wordmark-icon span { height: 60%; border-radius: 2px; background: #f3f4ee; }
.wordmark-icon span:nth-child(2) { height: 100%; background: #ed9b72; }
.wordmark-icon span:nth-child(3) { height: 78%; }
.topbar-note { display: flex; justify-content: center; align-items: center; gap: 8px; color: var(--muted); font-size: 12px; }
.live-dot { width: 7px; height: 7px; border-radius: 50%; background: #80a67b; box-shadow: 0 0 0 4px #e1ebdf; }
.topbar-score { display: flex; justify-content: flex-end; align-items: center; gap: 8px; font-size: 13px; font-weight: 700; }
.score-star { color: var(--coral); font-size: 19px; }
.score-label { color: var(--muted); font-weight: 500; }

.quiz-main { display: flex; width: min(100%, 1110px); flex: 1; flex-direction: column; justify-content: center; margin: 0 auto; padding: 45px 0 40px; }
.progress-section { margin-bottom: 34px; }
.progress-copy { display: flex; justify-content: space-between; align-items: center; margin-bottom: 11px; }
.eyebrow { margin: 0; color: var(--leaf); font-size: 10px; font-weight: 700; letter-spacing: .14em; text-transform: uppercase; }
.question-count { color: var(--ink); font-size: 12px; font-weight: 700; }
.question-count i { padding: 0 5px; color: #a6afa6; font-style: normal; font-weight: 400; }
.progress-track { height: 4px; overflow: hidden; border-radius: 9px; background: #dfe4db; }
.progress-track span { display: block; height: 100%; border-radius: inherit; background: var(--leaf); transition: width .45s ease; }

.question-layout { display: grid; grid-template-columns: minmax(0, 1.05fr) minmax(360px, .95fr); gap: clamp(42px, 7.2vw, 104px); align-items: center; animation: enter .45s ease both; }
.image-panel { min-width: 0; }
.image-meta { display: flex; align-items: center; gap: 9px; margin-bottom: 13px; color: var(--muted); font-size: 10px; font-weight: 600; letter-spacing: .09em; text-transform: uppercase; }
.image-meta span:nth-child(2) { color: var(--coral); }
.question-image { display: block; width: 100%; aspect-ratio: 1.42; object-fit: cover; border-radius: 5px; box-shadow: 0 18px 38px rgba(39, 57, 46, .10); }
.image-caption { display: flex; align-items: center; gap: 8px; margin-top: 14px; color: var(--muted); font-size: 11px; }
.caption-mark { display: grid; width: 19px; height: 19px; place-items: center; border: 1px solid #ccd5ca; border-radius: 50%; color: var(--leaf); font-size: 11px; }

.answer-panel { padding: 6px 0; }
.question-kicker { margin: 0 0 11px; color: var(--coral); font-size: 11px; font-weight: 700; letter-spacing: .08em; text-transform: uppercase; }
h1 { max-width: 470px; margin: 0; font-family: Georgia, serif; font-size: clamp(32px, 4vw, 48px); font-weight: 500; line-height: 1.12; }
.instruction { margin: 13px 0 24px; color: var(--muted); font-size: 13px; }
.choices { display: grid; gap: 9px; }
.choice-button { display: grid; min-height: 56px; grid-template-columns: 30px 1fr 20px; align-items: center; gap: 12px; padding: 0 15px; border: 1px solid #d9ded6; border-radius: 4px; background: rgba(255, 255, 255, .54); cursor: pointer; text-align: left; transition: border-color .18s ease, background .18s ease, transform .18s ease; }
.choice-button:hover:not(:disabled) { transform: translateX(3px); border-color: var(--leaf); background: #fff; }
.choice-button:focus-visible, .next-button:focus-visible, .wordmark:focus-visible { outline: 3px solid #e89a70; outline-offset: 3px; }
.choice-button:disabled { cursor: default; }
.choice-letter { display: grid; width: 26px; height: 26px; place-items: center; border-radius: 50%; background: #edf0e9; color: #65756a; font-size: 10px; font-weight: 700; }
.choice-text { font-size: 13px; font-weight: 600; }
.choice-mark { justify-self: end; color: #9aa49a; font-size: 13px; }
.choice-button.is-correct { border-color: #6e9c78; background: #eaf2e9; }
.choice-button.is-correct .choice-letter { background: #326b51; color: white; }
.choice-button.is-correct .choice-mark { color: #326b51; font-weight: 700; }
.choice-button.is-wrong { border-color: #dd8a72; background: #fbede7; }
.choice-button.is-wrong .choice-letter, .choice-button.is-wrong .choice-mark { background: #e97556; color: white; }
.answer-feedback { display: flex; align-items: center; gap: 9px; margin-top: 15px; color: var(--leaf); font-size: 12px; font-weight: 600; animation: enter .25s ease both; }
.feedback-icon { display: grid; width: 20px; height: 20px; place-items: center; border-radius: 50%; background: #dcebdd; font-size: 11px; }
.feedback-wrong { color: #a84c35; }
.feedback-wrong .feedback-icon { background: #f6ded5; }
.next-button { display: inline-flex; min-height: 47px; align-items: center; justify-content: space-between; gap: 28px; margin-top: 17px; padding: 0 18px; border: 0; border-radius: 4px; background: var(--ink); color: #fff; cursor: pointer; font-size: 12px; font-weight: 600; transition: background .18s ease, transform .18s ease; animation: enter .25s ease both; }
.next-button:hover { transform: translateY(-2px); background: var(--leaf); }
.next-button span { font-size: 17px; font-weight: 400; }

.result-screen { display: flex; min-height: 480px; flex-direction: column; align-items: center; justify-content: center; text-align: center; animation: enter .5s ease both; }
.result-orbit { display: grid; width: 92px; height: 92px; place-items: center; margin-bottom: 25px; border: 1px solid #c6d5c2; border-radius: 50%; color: var(--coral); font-size: 40px; animation: turn 18s linear infinite; }
.result-orbit span { animation: turn 18s linear infinite reverse; }
.result-screen h1 { margin-top: 13px; }
.result-copy { margin: 14px 0 22px; color: var(--muted); font-size: 14px; }
.result-copy strong { color: var(--ink); }
.result-score { display: flex; align-items: baseline; gap: 10px; color: var(--leaf); font-family: Georgia, serif; font-size: 34px; }
.result-score i { color: #a5afa4; font-family: 'Trebuchet MS', sans-serif; font-size: 17px; font-style: normal; }
.restart-button { margin-top: 22px; }
.footer { display: flex; min-height: 58px; align-items: center; justify-content: space-between; border-top: 1px solid var(--line); color: var(--muted); font-size: 10px; }
.footer i { padding: 0 4px; color: var(--coral); font-style: normal; }

@keyframes enter { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
@keyframes turn { to { transform: rotate(360deg); } }

@media (max-width: 760px) {
  .app-shell { padding: 0 6vw; }
  .topbar { min-height: 68px; grid-template-columns: 1fr auto; }
  .topbar-note { display: none; }
  .quiz-main { justify-content: flex-start; padding: 28px 0 30px; }
  .progress-section { margin-bottom: 25px; }
  .question-layout { grid-template-columns: 1fr; gap: 28px; }
  .question-image { aspect-ratio: 1.55; }
  h1 { max-width: 540px; font-size: clamp(32px, 9vw, 42px); }
  .instruction { margin-bottom: 18px; }
  .choice-button { min-height: 54px; }
  .result-screen { min-height: 65vh; }
}

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { scroll-behavior: auto !important; animation-duration: .01ms !important; animation-iteration-count: 1 !important; transition-duration: .01ms !important; }
}
</style>
