```vue
<script lang="ts" setup>
import {computed, ref} from 'vue'
import html2canvas from 'html2canvas'

type DateType = 'calm' | 'fun' | 'trust'

const step = ref(0)
const selectedDate = ref<DateType | null>(null)
const accepted = ref(false)
const shareCard = ref<HTMLElement | null>(null)

const noButtonStyle = ref({
  transform: 'translate(0px, 0px)',
})

const dateOptions = [
  {
    id: 'calm' as const,
    emoji: '🌙',
    title: 'Something calm',
    text: 'Relaxed, easy, and enough time to actually talk.',
  },
  {
    id: 'fun' as const,
    emoji: '🎯',
    title: 'Something fun',
    text: 'A little chaos, a little competition, definitely something to laugh about.',
  },
  {
    id: 'trust' as const,
    emoji: '✨',
    title: 'Trust me',
    text: 'No spoilers. I plan it, you just show up.',
  },
]

const selectedOption = computed(() =>
    dateOptions.find(option => option.id === selectedDate.value)
)

function next() {
  step.value++
}

function chooseDate(id: DateType) {
  selectedDate.value = id
}

function escapeNo() {
  const x = Math.floor(Math.random() * 180) - 90
  const y = Math.floor(Math.random() * 110) - 55

  noButtonStyle.value = {
    transform: `translate(${x}px, ${y}px)`,
  }
}

async function shareResult() {
  if (!shareCard.value) return

  try {
    const canvas = await html2canvas(shareCard.value, {
      scale: 2,
      backgroundColor: '#f7f9f7',
      useCORS: true,
    })

    canvas.toBlob(async blob => {
      if (!blob) return

      const file = new File(
          [blob],
          'sofia-date-approved.png',
          {type: 'image/png'}
      )

      if (
          navigator.share &&
          navigator.canShare &&
          navigator.canShare({files: [file]})
      ) {
        await navigator.share({
          files: [file],
          title: 'Date approved ♡',
          text: 'Official evidence 😌',
        })
      } else {
        const link = document.createElement('a')

        link.download = 'sofia-date-approved.png'
        link.href = canvas.toDataURL('image/png')

        document.body.appendChild(link)
        link.click()
        document.body.removeChild(link)
      }
    }, 'image/png')
  } catch (error) {
    console.error('Could not share result:', error)
  }
}
</script>

<template>
  <main class="page">
    <div class="blob blob-one"></div>
    <div class="blob blob-two"></div>

    <Transition mode="out-in" name="fade">

      <!-- INTRO -->
      <section
          v-if="step === 0"
          key="intro"
          class="screen"
      >
        <div class="card hero-card">
          <span class="eyebrow">
            Sofias Website — V2.0
          </span>

          <h1>
            So, Sofia...
          </h1>

          <p class="lead">
            Apparently making an entire website was not enough.
          </p>

          <p class="muted">
            Which is concerning, because this means I have now developed a sequel.
          </p>

          <button
              class="primary-button"
              @click="next"
          >
            Unfortunately, continue →
          </button>
        </div>
      </section>

      <!-- CALLBACK -->
      <section
          v-else-if="step === 1"
          key="callback"
          class="screen"
      >
        <div class="card">
          <span class="emoji">
            🧪
          </span>

          <h2>
            The experiment continues.
          </h2>

          <p>
            We’ve done the application process.
            We’ve reviewed the evidence.
            And you still keep closing your eyes 🙈🙈🙈
          </p>

          <p class="highlight">
            I think there is only one scientifically responsible next step.
          </p>

          <button
              class="primary-button"
              @click="next"
          >
            Conduct field research
          </button>
        </div>
      </section>

      <!-- DATE CHOICE -->
      <section
          v-else-if="step === 2"
          key="choice"
          class="screen"
      >
        <div class="card wide-card">
          <span class="eyebrow">
            SELECT DATE VIBE
          </span>

          <h2>
            Hypothetically...
          </h2>

          <p class="muted">
            If two highly qualified researchers were to go on a date,
            what kind of vibe should it have?
          </p>

          <div class="options">
            <button
                v-for="option in dateOptions"
                :key="option.id"
                :class="{ selected: selectedDate === option.id }"
                class="option-card"
                @click="chooseDate(option.id)"
            >
              <span class="option-emoji">
                {{ option.emoji }}
              </span>

              <strong>
                {{ option.title }}
              </strong>

              <small>
                {{ option.text }}
              </small>
            </button>
          </div>

          <Transition name="fade">
            <button
                v-if="selectedDate"
                class="primary-button"
                @click="next"
            >
              Solid choice →
            </button>
          </Transition>
        </div>
      </section>

      <!-- REVEAL -->
      <section
          v-else-if="step === 3"
          key="reveal"
          class="screen"
      >
        <div class="card">
          <span class="emoji">
            {{ selectedOption?.emoji }}
          </span>

          <h2>
            Interesting choice.
          </h2>

          <p>
            So technically, you have already participated in planning it.
          </p>

          <p class="muted">
            Which makes the next question significantly harder to escape.
          </p>

          <button
              class="primary-button"
              @click="next"
          >
            I don’t like where this is going
          </button>
        </div>
      </section>

      <!-- FINAL QUESTION -->
      <section
          v-else-if="step === 4 && !accepted"
          key="question"
          class="screen"
      >
        <div class="card question-card">
          <span class="eyebrow">
            FINAL QUESTION
          </span>

          <div class="heart">
            ♡
          </div>

          <h1 class="date-question">
            Sofia,<br/>
            would you like to go on a date with me?
          </h1>

          <p class="muted">
            {{ selectedOption?.title }} edition.
          </p>

          <div class="answer-buttons">
            <button
                class="yes-button"
                @click="accepted = true"
            >
              Yes :)
            </button>

            <button
                :style="noButtonStyle"
                class="no-button"
                @pointerdown="escapeNo"
                @pointerenter="escapeNo"
            >
              No
            </button>
          </div>
        </div>
      </section>

      <!-- SUCCESS -->
      <section
          v-else
          key="success"
          class="screen"
      >
        <div class="success-wrapper">

          <!-- THIS PART BECOMES THE IMAGE -->
          <div
              ref="shareCard"
              class="card success-card"
          >
            <div class="success-icon">
              ✓
            </div>

            <span class="eyebrow">
              APPLICATION STATUS
            </span>

            <h1>
              Date approved.
            </h1>

            <p class="lead">
              Excellent decision.
            </p>

            <div
                v-if="selectedOption"
                class="shared-choice"
            >
              <span class="shared-choice-emoji">
                {{ selectedOption.emoji }}
              </span>

              <strong>
                {{ selectedOption.title }}
              </strong>

              <p>
                {{ selectedOption.text }}
              </p>
            </div>

            <div class="mini-note">
              Official evidence has been recorded for completely legal purposes.
            </div>

            <div class="final-line">
              See you soon, Sofia ♡
            </div>
          </div>

          <!-- BUTTON IS OUTSIDE SCREENSHOT -->
          <button
              class="primary-button share-button"
              @click="shareResult"
          >
            Share the result ↗
          </button>

        </div>
      </section>

    </Transition>
  </main>
</template>
```