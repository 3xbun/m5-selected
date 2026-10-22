<script setup>
import { computed, onMounted, onUnmounted, ref } from 'vue'
import dayjs from 'dayjs'
import { subjects } from './SelectedSubject.json'
import FlipCountdown from './components/FlipCountdown.vue'

const TARGET_TIME = dayjs('2026-10-22T11:36:30')
const ALL_TRACKS = 'ทุกแผนการเรียน'

const targetDateLong = new Intl.DateTimeFormat('th-TH', {
  day: 'numeric',
  month: 'long',
  year: 'numeric',
}).format(TARGET_TIME.toDate())
const targetDateShort = new Intl.DateTimeFormat('th-TH', {
  day: 'numeric',
  month: 'short',
}).format(TARGET_TIME.toDate())

const openAtLong = `${targetDateLong} เวลา ${TARGET_TIME.format('HH:mm')} น.`
const openAtShort = `${targetDateShort} ${TARGET_TIME.format('HH:mm')} น.`

/* ------------------------------------------------------------------ */
/* Countdown                                                          */
/* ------------------------------------------------------------------ */

const now = ref(dayjs())
let ticker = null

const isOpen = computed(() => TARGET_TIME.diff(now.value) <= 0)

function pad(value) {
  return String(value).padStart(2, '0')
}

const timeLeft = computed(() => {
  const totalSeconds = Math.max(Math.floor(TARGET_TIME.diff(now.value) / 1000), 0)

  return {
    totalSeconds,
    days: pad(Math.floor(totalSeconds / 86400)),
    hours: pad(Math.floor((totalSeconds % 86400) / 3600)),
    minutes: pad(Math.floor((totalSeconds % 3600) / 60)),
    seconds: pad(totalSeconds % 60),
  }
})

const showDays = computed(() => timeLeft.value.totalSeconds >= 86400)

const countdownLabel = computed(() => {
  if (isOpen.value) return 'เปิดลงทะเบียนแล้ว'

  const parts = []
  if (showDays.value) parts.push(`${Number(timeLeft.value.days)} วัน`)
  parts.push(
    `${Number(timeLeft.value.hours)} ชั่วโมง`,
    `${Number(timeLeft.value.minutes)} นาที`,
    `${Number(timeLeft.value.seconds)} วินาที`,
  )

  return `เหลือเวลาอีก ${parts.join(' ')} ก่อนเปิดลงทะเบียน`
})

onMounted(() => {
  if (isOpen.value) return

  ticker = window.setInterval(() => {
    now.value = dayjs()

    if (isOpen.value) {
      window.clearInterval(ticker)
      ticker = null
    }
  }, 1000)
})

onUnmounted(() => {
  if (ticker) window.clearInterval(ticker)
})

/* ------------------------------------------------------------------ */
/* Eligibility                                                        */
/* ------------------------------------------------------------------ */

function isOpenToAll(condition) {
  return condition === ALL_TRACKS
}

function hasFormLink(subject) {
  return Boolean(subject.link) && subject.link !== 'https://google.com'
}

/* ------------------------------------------------------------------ */
/* Images                                                             */
/* ------------------------------------------------------------------ */

const brokenImages = ref(new Set())

function onImageError(id) {
  const next = new Set(brokenImages.value)
  next.add(id)
  brokenImages.value = next
}
</script>

<template>
  <div id="top" class="page">
    <header class="site-header">
      <div class="container site-header__inner">
        <a class="brand" href="#top" aria-label="กลับขึ้นด้านบน">
          <span class="brand__mark" aria-hidden="true">M6</span>
          <span class="brand__name">เลือกเสรี <small>มัธยมศึกษาปีที่ 6</small></span>
        </a>
        <span class="year-chip">ปีการศึกษา 2569</span>
      </div>
    </header>

    <main>
      <section class="hero container" aria-labelledby="page-title">
        <p class="eyebrow">เลือกเรียนตามความสนใจ</p>
        <h1 id="page-title">วิชาเลือกเสรี</h1>
        <p class="hero__lead">
          เลือกวิชาที่ใช่สำหรับเทอมนี้ ตรวจสอบแผนการเรียนและจำนวนที่รับให้ครบก่อนลงทะเบียน
        </p>

        <div class="status-card" :class="{ 'status-card--open': isOpen }">
          <div class="status-card__info">
            <span
              class="status-pill"
              :class="isOpen ? 'status-pill--open' : 'status-pill--waiting'"
            >
              <span class="status-pill__dot" aria-hidden="true"></span>
              {{ isOpen ? 'เปิดลงทะเบียนแล้ว' : 'ยังไม่เปิดลงทะเบียน' }}
            </span>

            <p v-if="!isOpen" class="status-card__note">
              เปิดลงทะเบียน <strong>{{ openAtLong }}</strong>
            </p>
            <p v-else class="status-card__note">
              เลือกวิชาที่สนใจ แล้วกด <strong>“ลงทะเบียน”</strong> ในวิชานั้นได้เลย
            </p>
          </div>

          <div v-if="!isOpen" class="status-card__timer">
            <FlipCountdown
              :days="timeLeft.days"
              :hours="timeLeft.hours"
              :minutes="timeLeft.minutes"
              :seconds="timeLeft.seconds"
              :show-days="showDays"
              :label="countdownLabel"
            />
          </div>

          <div v-else class="open-mark" aria-hidden="true">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8">
              <circle cx="12" cy="12" r="9" />
              <path d="m8.5 12 2.5 2.5 4.5-5" stroke-linecap="round" stroke-linejoin="round" />
            </svg>
          </div>
        </div>

        <p class="sr-only" aria-live="polite">
          {{ isOpen ? 'เปิดลงทะเบียนแล้ว' : '' }}
        </p>
      </section>

      <section class="catalog container" aria-labelledby="catalog-title">
        <h2 id="catalog-title" class="sr-only">รายวิชาเลือกเสรี</h2>

        <div class="results-bar">
          <p><strong>{{ subjects.length }}</strong> วิชาที่เปิดให้เลือก</p>
        </div>

        <div class="subject-grid">
          <article
            v-for="(subject, index) in subjects"
            :key="subject.id"
            class="subject-card"
            :style="{ '--i': index }"
            :aria-labelledby="`subject-title-${subject.id}`"
          >
            <p
              class="condition-banner"
              :class="
                isOpenToAll(subject.condition)
                  ? 'condition-banner--all'
                  : 'condition-banner--specific'
              "
            >
              <svg
                viewBox="0 0 24 24"
                width="17"
                height="17"
                fill="none"
                stroke="currentColor"
                stroke-width="2"
                aria-hidden="true"
              >
                <path d="M12 3l7 3v5.5c0 4.3-3 7.6-7 9.5-4-1.9-7-5.2-7-9.5V6l7-3Z" stroke-linejoin="round" />
                <path d="m9 11.5 2 2 4-4" stroke-linecap="round" stroke-linejoin="round" />
              </svg>
              <span class="condition-banner__text">
                <span class="condition-banner__label">เงื่อนไขการรับ</span>
                <span class="condition-banner__value">{{ subject.condition }}</span>
              </span>
            </p>

            <div class="card__media">
              <svg
                class="card__fallback"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="1.4"
                aria-hidden="true"
              >
                <path d="M4 5.5A2.5 2.5 0 0 1 6.5 3H20v18H6.5A2.5 2.5 0 0 1 4 18.5v-13Z" />
                <path d="M4 18.5A2.5 2.5 0 0 1 6.5 16H20" />
              </svg>

              <img
                v-if="!brokenImages.has(subject.id)"
                :src="subject.image"
                :alt="`สัญลักษณ์ประจำวิชา ${subject.title}`"
                loading="lazy"
                decoding="async"
                @error="onImageError(subject.id)"
              />

              <span v-if="subject.limit" class="tag tag--seats">
                <svg
                  viewBox="0 0 24 24"
                  width="13"
                  height="13"
                  fill="none"
                  stroke="currentColor"
                  stroke-width="1.8"
                  aria-hidden="true"
                >
                  <path d="M16 20v-1.5a4.5 4.5 0 0 0-4.5-4.5h-4A4.5 4.5 0 0 0 3 18.5V20" stroke-linecap="round" />
                  <circle cx="9.5" cy="8" r="3.5" />
                  <path d="M17 4.5a3.5 3.5 0 0 1 0 7M21 20v-1.5a4.5 4.5 0 0 0-3-4.2" stroke-linecap="round" />
                </svg>
                รับ {{ subject.limit }} คน
              </span>
            </div>

            <div class="card__body">
              <h3 :id="`subject-title-${subject.id}`" class="card__title">
                {{ subject.title }}
              </h3>

              <p v-if="subject.desc" class="card__desc">{{ subject.desc }}</p>

              <ul class="meta">
                <li>
                  <svg
                    viewBox="0 0 24 24"
                    width="14"
                    height="14"
                    fill="none"
                    stroke="currentColor"
                    stroke-width="1.8"
                    aria-hidden="true"
                  >
                    <path d="M20 21a8 8 0 0 0-16 0" stroke-linecap="round" />
                    <circle cx="12" cy="8" r="4" />
                  </svg>
                  <span>{{ subject.teacher }}</span>
                </li>
                <li>
                  <svg
                    viewBox="0 0 24 24"
                    width="14"
                    height="14"
                    fill="none"
                    stroke="currentColor"
                    stroke-width="1.8"
                    aria-hidden="true"
                  >
                    <path d="M12 21s7-5.4 7-11a7 7 0 1 0-14 0c0 5.6 7 11 7 11Z" stroke-linejoin="round" />
                    <circle cx="12" cy="10" r="2.5" />
                  </svg>
                  <span>{{ subject.room }}</span>
                </li>
              </ul>
            </div>

            <div class="card__footer">
              <a
                v-if="isOpen && hasFormLink(subject)"
                class="btn"
                :href="subject.link"
                target="_blank"
                rel="noopener noreferrer"
                :aria-label="`ลงทะเบียนวิชา ${subject.title} (เปิดในแท็บใหม่)`"
              >
                ลงทะเบียน
                <svg
                  viewBox="0 0 24 24"
                  width="16"
                  height="16"
                  fill="none"
                  stroke="currentColor"
                  stroke-width="1.8"
                  aria-hidden="true"
                >
                  <path d="M5 12h14M13 6l6 6-6 6" stroke-linecap="round" stroke-linejoin="round" />
                </svg>
              </a>

              <button v-else type="button" class="btn btn--closed" disabled>
                <svg
                  viewBox="0 0 24 24"
                  width="15"
                  height="15"
                  fill="none"
                  stroke="currentColor"
                  stroke-width="1.8"
                  aria-hidden="true"
                >
                  <rect x="4.5" y="10" width="15" height="10" rx="2.5" />
                  <path d="M8 10V7a4 4 0 0 1 8 0v3" stroke-linecap="round" />
                </svg>
                {{ isOpen ? 'ยังไม่มีแบบฟอร์ม' : `เปิด ${openAtShort}` }}
              </button>
            </div>
          </article>
        </div>

        <p v-if="isOpen" class="catalog__note">
          การลงทะเบียนจะเปิดแบบฟอร์มในแท็บใหม่
        </p>
      </section>
    </main>

    <footer class="site-footer">
      <div class="container site-footer__inner">
        <p>สงสัยเกี่ยวกับการเลือกวิชา? สอบถามครูที่ปรึกษาได้เลย</p>
        <p class="site-footer__meta">วิชาเลือกเสรี · มัธยมศึกษาปีที่ 6 · ปีการศึกษา 2569</p>
      </div>
    </footer>
  </div>
</template>

<style lang="css" scoped>
.page {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.container {
  width: min(100% - 40px, var(--max-width));
  margin-inline: auto;
}

/* Header ------------------------------------------------------------ */

.site-header {
  position: relative;
  z-index: 2;
  border-bottom: 1px solid rgba(255, 255, 255, 0.06);
  background: rgba(7, 11, 22, 0.35);
  backdrop-filter: blur(10px);
}

.site-header__inner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  padding-block: 16px;
}

.brand {
  display: inline-flex;
  align-items: center;
  gap: 12px;
  border-radius: 12px;
}

.brand__mark {
  display: grid;
  place-items: center;
  width: 40px;
  height: 40px;
  border-radius: 12px;
  border: 1px solid rgba(124, 155, 255, 0.35);
  background: linear-gradient(135deg, rgba(91, 124, 250, 0.28), rgba(139, 92, 246, 0.18));
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.12);
  color: #c7d4ff;
  font-size: 0.85rem;
  font-weight: 800;
}

.brand__name {
  display: flex;
  align-items: baseline;
  gap: 8px;
  font-size: 0.95rem;
  font-weight: 600;
}

.brand__name small {
  color: var(--muted);
  font-size: 0.78rem;
  font-weight: 500;
}

.year-chip {
  padding: 5px 12px;
  border: 1px solid var(--border);
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.03);
  color: var(--muted-strong);
  font-size: 0.78rem;
  white-space: nowrap;
}

/* Hero -------------------------------------------------------------- */

.hero {
  padding-top: clamp(40px, 7vw, 78px);
  text-align: center;
}

.eyebrow {
  display: inline-flex;
  align-items: center;
  padding: 4px 12px;
  border: 1px solid var(--border);
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.03);
  color: #aab9d6;
  font-size: 0.76rem;
  font-weight: 600;
}

h1 {
  margin-top: 16px;
  font-size: clamp(2.4rem, 6.5vw, 4rem);
  line-height: 1.22;
  font-weight: 800;
  letter-spacing: -0.01em;
  text-wrap: balance;
}

.hero__lead {
  max-width: 46ch;
  margin: 12px auto 0;
  color: var(--muted);
  font-size: clamp(0.92rem, 2.2vw, 1.02rem);
  line-height: 1.75;
  text-wrap: pretty;
}

/* Status card ------------------------------------------------------- */

.status-card {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: clamp(16px, 3vw, 36px);
  max-width: 880px;
  margin: clamp(26px, 5vw, 40px) auto 0;
  padding: clamp(18px, 3vw, 26px) clamp(18px, 3vw, 30px);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  background:
    linear-gradient(
      140deg,
      rgba(255, 255, 255, 0.075),
      rgba(255, 255, 255, 0.025) 55%,
      rgba(91, 124, 250, 0.07)
    );
  box-shadow:
    0 30px 60px -40px rgba(0, 0, 0, 0.9),
    inset 0 1px 0 rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(12px);
  text-align: left;
  flex-wrap: wrap;
}

.status-card--open {
  border-color: rgba(52, 211, 153, 0.22);
  background:
    linear-gradient(
      140deg,
      rgba(52, 211, 153, 0.08),
      rgba(255, 255, 255, 0.025) 60%,
      rgba(52, 211, 153, 0.04)
    );
}

.status-card__info {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 10px;
}

.status-pill {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 4px 12px;
  border: 1px solid;
  border-radius: 999px;
  font-size: 0.78rem;
  font-weight: 600;
}

.status-pill__dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: currentColor;
}

.status-pill--waiting {
  border-color: rgba(251, 191, 36, 0.28);
  background: rgba(251, 191, 36, 0.09);
  color: #fbbf24;
}

.status-pill--waiting .status-pill__dot {
  animation: blink 1.8s ease-in-out infinite;
}

.status-pill--open {
  border-color: rgba(52, 211, 153, 0.3);
  background: rgba(52, 211, 153, 0.1);
  color: var(--success);
}

.status-card__note {
  color: var(--muted-strong);
  font-size: 0.9rem;
  line-height: 1.65;
}

.status-card__note strong {
  color: #fff;
  font-weight: 600;
}

.status-card__timer {
  display: flex;
  justify-content: flex-end;
  flex: 0 1 auto;
}

.open-mark {
  display: grid;
  place-items: center;
  width: 58px;
  height: 58px;
  border: 1px solid rgba(52, 211, 153, 0.3);
  border-radius: 50%;
  background: rgba(52, 211, 153, 0.1);
  color: var(--success);
  animation: pop 0.4s cubic-bezier(0.2, 0.7, 0.3, 1) both;
}

.open-mark svg {
  width: 30px;
  height: 30px;
}

/* Catalog ----------------------------------------------------------- */

.catalog {
  flex: 1;
  padding-top: clamp(32px, 5vw, 56px);
  padding-bottom: 64px;
}

.results-bar {
  margin: 0 4px 16px;
  color: var(--muted);
  font-size: 0.86rem;
}

.results-bar strong {
  color: #fff;
  font-weight: 600;
}

/* Grid & cards ------------------------------------------------------ */

.subject-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(min(100%, 280px), 1fr));
  gap: 18px;
}

.subject-card {
  display: flex;
  flex-direction: column;
  overflow: hidden;
  border: 1px solid var(--border);
  border-radius: 20px;
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.055), rgba(255, 255, 255, 0.025));
  box-shadow: 0 18px 40px -28px rgba(0, 0, 0, 0.85);
  animation: rise 0.5s cubic-bezier(0.2, 0.7, 0.3, 1) both;
  animation-delay: calc(var(--i, 0) * 45ms);
  transition: transform 0.25s ease, border-color 0.25s ease, box-shadow 0.25s ease;
}

.subject-card:hover {
  transform: translateY(-4px);
  border-color: rgba(124, 155, 255, 0.35);
  box-shadow:
    0 30px 60px -30px rgba(0, 0, 0, 0.85),
    0 0 0 1px rgba(124, 155, 255, 0.12);
}

.card__media {
  position: relative;
  display: grid;
  place-items: center;
  aspect-ratio: 16 / 10;
  background:
    radial-gradient(circle at 50% 30%, rgba(124, 155, 255, 0.18), transparent 65%),
    linear-gradient(150deg, #13203a, #0b1322);
}

.card__media img {
  position: relative;
  z-index: 1;
  width: 62%;
  height: 62%;
  object-fit: contain;
  filter: drop-shadow(0 12px 20px rgba(0, 0, 0, 0.35));
  transition: transform 0.35s ease;
}

.subject-card:hover .card__media img {
  transform: scale(1.06) translateY(-2px);
}

.card__fallback {
  position: absolute;
  width: 62px;
  height: 62px;
  color: rgba(255, 255, 255, 0.14);
}

.tag {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 3px 9px;
  border: 1px solid rgba(255, 255, 255, 0.14);
  border-radius: 999px;
  background: rgba(7, 11, 22, 0.72);
  color: #e8eeff;
  font-size: 0.72rem;
  line-height: 1.6;
  white-space: nowrap;
  backdrop-filter: blur(6px);
}

.tag--seats {
  position: absolute;
  top: 10px;
  right: 10px;
  z-index: 2;
}

.card__body {
  display: flex;
  flex: 1;
  flex-direction: column;
  gap: 10px;
  padding: 16px 18px 4px;
}

.condition-banner {
  display: flex;
  align-items: center;
  gap: 10px;
  min-height: 4.75rem;
  padding: 10px 16px;
  color: #fff;
  line-height: 1.4;
}

.condition-banner > svg {
  flex: none;
  box-sizing: content-box;
  padding: 4px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.16);
}

.condition-banner__text {
  display: flex;
  flex-direction: column;
}

.condition-banner__label {
  font-size: 0.64rem;
  font-weight: 600;
  opacity: 0.82;
}

.condition-banner__value {
  font-size: 0.86rem;
  font-weight: 700;
  line-height: 1.45;
}

.condition-banner--all {
  border-bottom: 1px solid rgba(255, 255, 255, 0.16);
  background: linear-gradient(135deg, #065f46, #059669);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.15);
}

.condition-banner--specific {
  border-bottom: 1px solid rgba(255, 255, 255, 0.18);
  background: linear-gradient(135deg, #881337, #e11d48);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.18);
}

.card__title {
  display: -webkit-box;
  overflow: hidden;
  color: #fff;
  font-size: 1.02rem;
  font-weight: 700;
  line-height: 1.5;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
}

.card__desc {
  display: -webkit-box;
  overflow: hidden;
  color: var(--muted);
  font-size: 0.82rem;
  line-height: 1.6;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
}

.meta {
  display: flex;
  flex-direction: column;
  gap: 6px;
  margin-top: 2px;
  color: var(--muted-strong);
  font-size: 0.82rem;
}

.meta li {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  line-height: 1.55;
}

.meta svg {
  flex: none;
  margin-top: 3px;
  color: #8296b8;
}

.card__footer {
  padding: 14px 18px 18px;
}

/* Buttons ----------------------------------------------------------- */

.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  width: 100%;
  min-height: 44px;
  padding: 10px 16px;
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 12px;
  background: linear-gradient(135deg, var(--accent-strong), var(--accent-2));
  box-shadow: 0 10px 24px -12px rgba(91, 124, 250, 0.85);
  color: #fff;
  font-size: 0.9rem;
  font-weight: 600;
  transition: filter 0.2s ease, transform 0.2s ease, box-shadow 0.2s ease;
}

.btn:hover {
  filter: brightness(1.08);
  transform: translateY(-1px);
}

.btn:active {
  transform: translateY(0);
}

.btn--closed,
.btn--closed:disabled {
  border: 1px dashed var(--border-strong);
  background: rgba(255, 255, 255, 0.035);
  box-shadow: none;
  color: var(--muted);
  font-weight: 500;
  cursor: not-allowed;
}

.btn--closed:hover {
  filter: none;
  transform: none;
}

.catalog__note {
  margin-top: 20px;
  color: #7f8ea9;
  font-size: 0.8rem;
  text-align: center;
}

/* Footer ------------------------------------------------------------ */

.site-footer {
  border-top: 1px solid rgba(255, 255, 255, 0.06);
  background: rgba(5, 8, 15, 0.5);
  padding-block: 26px;
}

.site-footer__inner {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  gap: 6px 18px;
  color: var(--muted);
  font-size: 0.8rem;
  line-height: 1.7;
}

.site-footer__meta {
  color: #6f7e97;
}

/* Animations -------------------------------------------------------- */

@keyframes rise {
  from {
    opacity: 0;
    transform: translateY(14px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes blink {
  0%,
  100% {
    opacity: 1;
    box-shadow: 0 0 0 4px rgba(251, 191, 36, 0.12);
  }
  50% {
    opacity: 0.55;
    box-shadow: 0 0 0 6px rgba(251, 191, 36, 0.05);
  }
}

@keyframes pop {
  from {
    opacity: 0;
    transform: scale(0.8);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

/* Responsive -------------------------------------------------------- */

@media (max-width: 640px) {
  .container {
    width: min(100% - 32px, var(--max-width));
  }

  .brand__name small {
    display: none;
  }

  .status-card {
    flex-direction: column;
    align-items: stretch;
    text-align: left;
  }

  .status-card__timer {
    justify-content: center;
    width: 100%;
    margin-top: 4px;
  }

  .open-mark {
    align-self: center;
  }

  .hero {
    padding-top: 34px;
  }

  .catalog {
    padding-bottom: 44px;
  }

  .site-footer__inner {
    flex-direction: column;
  }

  .subject-grid {
    gap: 14px;
  }
}

@media (max-width: 420px) {
  .year-chip {
    padding-inline: 9px;
    font-size: 0.72rem;
  }

  .status-card {
    padding: 16px;
  }
}
</style>
