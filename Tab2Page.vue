<template>
  <ion-page>
    <ion-header class="ion-no-border">
      <ion-toolbar class="main-toolbar">
        <div class="brand">
          <span class="brand-mark">AI</span>
          <span class="brand-name">Smart Quiz - คลังข้อสอบ</span>
        </div>
      </ion-toolbar>
    </ion-header>

    <ion-content class="quiz-page" :fullscreen="true">
      <!-- HERO -->
      <section class="hero">
        <div class="hero-grid"></div>
        <div class="hero-inner">
          <h1>คลังข้อสอบสะสม</h1>
          <p>รวมข้อสอบและสรุปเนื้อหาที่ AI เคยสร้างไว้ แยกตามรายวิชา ค้นหาง่าย ทบทวนได้ตลอดเวลา</p>

          <!-- Search & Filter Bar -->
          <div class="search-box">
            <svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round">
              <circle cx="11" cy="11" r="8" />
              <path d="M21 21l-4.35-4.35" />
            </svg>
            <input
              type="text"
              v-model="searchQuery"
              placeholder="ค้นหาชื่อข้อสอบ, เนื้อหา หรือคีย์เวิร์ด..."
            />
            <button v-if="searchQuery" class="clear-search" @click="searchQuery = ''">×</button>
          </div>
        </div>
      </section>

      <div class="content-wrapper">
        <!-- SUBJECT FILTER TABS -->
        <div class="subject-chips-scroll">
          <button
            v-for="sub in subjects"
            :key="sub"
            class="subject-chip"
            :class="{ active: selectedSubject === sub }"
            @click="selectedSubject = sub"
          >
            {{ sub }}
          </button>
        </div>

        <!-- STATS / COUNT BAR -->
        <div class="stats-bar">
          <span>พบทั้งหมด <strong>{{ filteredQuizzes.length }}</strong> ชุดข้อสอบ</span>
          <span class="sort-label">เรียงตาม: ล่าสุด</span>
        </div>

        <!-- QUIZ LIST -->
        <div v-if="filteredQuizzes.length > 0" class="quiz-list">
          <div v-for="quiz in filteredQuizzes" :key="quiz.id" class="panel quiz-card">
            <div class="quiz-card-header">
              <span class="subject-badge">{{ quiz.subject }}</span>
              <span class="date-badge">{{ quiz.date }}</span>
            </div>

            <h3 class="quiz-title">{{ quiz.title }}</h3>
            <p class="quiz-desc">{{ quiz.summary }}</p>

            <div class="quiz-meta-info">
              <span>🎯 {{ quiz.difficulty }}</span>
              <span>📝 {{ quiz.count }} ข้อ</span>
              <span>🏷️ {{ quiz.type }}</span>
            </div>

            <div class="btn-row">
              <ion-button class="btn-main" @click="viewQuizDetails(quiz)">
                ดูข้อสอบ / เฉลย
              </ion-button>
              <ion-button class="btn-ghost" fill="clear" @click="copyQuiz(quiz)">
                คัดลอก
              </ion-button>
            </div>
          </div>
        </div>

        <!-- EMPTY STATE -->
        <div v-else class="panel empty-state">
          <div class="ph-icon">
            <svg viewBox="0 0 24 24" width="30" height="30" fill="none" stroke="currentColor" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round">
              <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z" />
              <polyline points="14 2 14 8 20 8" />
              <line x1="16" y1="13" x2="8" y2="13" />
              <line x1="16" y1="17" x2="8" y2="17" />
              <polyline points="10 9 9 9 8 9" />
            </svg>
          </div>
          <h3>ไม่พบข้อสอบในหมวดหมู่นี้</h3>
          <p>ลองเปลี่ยนคำค้นหา หรือสร้างข้อสอบใหม่จากหน้าแรกได้เลย</p>
        </div>

        <footer>
          <p>Nexgen</p>
        </footer>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonContent,
  IonButton
} from '@ionic/vue';

// ================================
// SUBJECTS & FILTER DATA
// ================================

const selectedSubject = ref('ทั้งหมด');
const searchQuery = ref('');

const subjects = ['ทั้งหมด', 'วิทยาศาสตร์', 'คณิตศาสตร์', 'ประวัติศาสตร์', 'ภาษาอังกฤษ', 'คอมพิวเตอร์'];

// ตัวอย่างข้อมูลจำลองประวัติคลังข้อสอบ
const quizList = ref([
  {
    id: 1,
    subject: 'วิทยาศาสตร์',
    title: 'เซลล์พืชและการสังเคราะห์ด้วยแสง',
    summary: 'สรุปองค์ประกอบภายในเซลล์พืช หน้าที่ของคลอโรพลาสต์ และกระบวนการเปลี่ยนพลังงานแสงเป็นพลังงานเคมี',
    difficulty: 'ปานกลาง',
    count: 10,
    type: 'ปรนัย 4 ตัวเลือก',
    date: '10 พ.ย. 2569',
    content: 'เนื้อหาข้อสอบและเฉลยจำลองชุดที่ 1...'
  },
  {
    id: 2,
    subject: 'คอมพิวเตอร์',
    title: 'พื้นฐานการเขียนโปรแกรม Python และโครงสร้างข้อมูล',
    summary: 'ความรู้เบื้องต้นเกี่ยวกับตัวแปร เงื่อนไข (If-Else), ลูป (Loop) และการจัดการ List เบื้องต้น',
    difficulty: 'ยาก',
    count: 15,
    type: 'ผสมปรนัยและอัตนัย',
    date: '8 พ.ย. 2569',
    content: 'เนื้อหาข้อสอบและเฉลยจำลองชุดที่ 2...'
  },
  {
    id: 3,
    subject: 'ประวัติศาสตร์',
    title: 'เหตุการณ์สำคัญสมัยอยุธยาตอนกลาง',
    summary: 'สรุปเหตุการณ์ทางประวัติศาสตร์ การค้าขายกับต่างชาติ และความสัมพันธ์ระหว่างอาณาจักร',
    difficulty: 'ง่าย',
    count: 5,
    type: 'ปรนัย 4 ตัวเลือก',
    date: '5 พ.ย. 2569',
    content: 'เนื้อหาข้อสอบและเฉลยจำลองชุดที่ 3...'
  }
]);

// กรองข้อมูลตามวิชาที่เลือกและคำค้นหา
const filteredQuizzes = computed(() => {
  return quizList.value.filter(quiz => {
    const matchSubject = selectedSubject.value === 'ทั้งหมด' || quiz.subject === selectedSubject.value;
    const matchSearch = quiz.title.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
                        quiz.summary.toLowerCase().includes(searchQuery.value.toLowerCase());
    return matchSubject && matchSearch;
  });
});

// ================================
// ACTIONS
// ================================

function viewQuizDetails(quiz: any) {
  alert(`เปิดดูรายละเอียดข้อสอบ: ${quiz.title}`);
  // สามารถเขียน Logic เปิด Modal หรือนำทางไปยังหน้าแสดงข้อสอบเต็มรูปแบบได้ตรงนี้
}

async function copyQuiz(quiz: any) {
  try {
    await navigator.clipboard.writeText(`${quiz.title}\n${quiz.summary}\n${quiz.content}`);
    alert('คัดลอกข้อสอบเรียบร้อยแล้ว! 📋');
  } catch {
    alert('ไม่สามารถคัดลอกได้');
  }
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Prompt:wght@400;500;600;700&display=swap');

/* ---------- TOKENS (โทนม่วง) ---------- */
.quiz-page,
.main-toolbar {
  --ink: #1d1530;
  --ink-soft: #6b6280;
  --line: #ebe5f5;
  --surface: #ffffff;
  --canvas: #f5f2fb;
  --violet: #7546a8;
  --violet-deep: #5b2f93;
  --violet-tint: #f1eaff;
  --amber: #ffb938;
  --night: #25103f;
  --radius-lg: 28px;
  --radius-md: 18px;
  --radius-sm: 12px;
  font-family: 'Prompt', 'Noto Sans Thai', system-ui, sans-serif;
}

.quiz-page {
  --background: var(--canvas);
  color: var(--ink);
}

/* ---------- HEADER ---------- */
ion-header {
  background: var(--night);
}

.main-toolbar {
  --background: var(--night);
  --border-width: 0;
  --min-height: 54px;
}

.brand {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 0 18px;
  color: #fff;
}

.brand-mark {
  display: grid;
  place-items: center;
  width: 30px;
  height: 30px;
  border-radius: 10px;
  font-size: 13px;
  font-weight: 700;
  color: var(--night);
  background: var(--amber);
}

.brand-name {
  font-size: 16px;
  font-weight: 600;
}

/* ---------- HERO & SEARCH ---------- */
.hero {
  position: relative;
  overflow: hidden;
  padding: 26px 20px 48px;
  color: #fff;
  background:
    radial-gradient(420px 260px at 90% 0%, rgba(255, 185, 56, 0.22), transparent 70%),
    radial-gradient(500px 320px at 0% 100%, rgba(150, 90, 225, 0.6), transparent 70%),
    var(--night);
  border-radius: 0 0 40px 40px;
}

.hero-grid {
  position: absolute;
  inset: 0;
  background-image:
    linear-gradient(rgba(255, 255, 255, 0.05) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255, 255, 255, 0.05) 1px, transparent 1px);
  background-size: 28px 28px;
  mask-image: linear-gradient(to bottom, #000, transparent 85%);
  -webkit-mask-image: linear-gradient(to bottom, #000, transparent 85%);
  pointer-events: none;
}

.hero-inner {
  position: relative;
  max-width: 650px;
  margin: 0 auto;
}

.hero h1 {
  margin: 8px 0 6px;
  font-size: 28px;
  font-weight: 700;
}

.hero p {
  margin: 0 0 18px;
  font-size: 13.5px;
  line-height: 1.6;
  color: rgba(255, 255, 255, 0.76);
}

/* Search Input inside Hero */
.search-box {
  position: relative;
  display: flex;
  align-items: center;
  background: rgba(255, 255, 255, 0.12);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 16px;
  padding: 0 14px;
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
}

.search-box:focus-within {
  border-color: var(--amber);
}

.search-box svg {
  color: rgba(255, 255, 255, 0.6);
  flex-shrink: 0;
}

.search-box input {
  width: 100%;
  background: transparent;
  border: none;
  outline: none;
  padding: 12px 10px;
  font: inherit;
  font-size: 14px;
  color: #fff;
}

.search-box input::placeholder {
  color: rgba(255, 255, 255, 0.5);
}

.clear-search {
  background: none;
  border: none;
  color: #fff;
  font-size: 18px;
  cursor: pointer;
  padding: 0 4px;
}

/* ---------- CONTENT & LAYOUT ---------- */
.content-wrapper {
  max-width: 650px;
  margin: -20px auto 0;
  padding: 0 14px 28px;
  position: relative;
}

/* Subject Chips Horizontal Scroll */
.subject-chips-scroll {
  display: flex;
  gap: 8px;
  overflow-x: auto;
  padding: 4px 2px 14px;
  scrollbar-width: none;
}

.subject-chips-scroll::-webkit-scrollbar {
  display: none;
}

.subject-chip {
  flex-shrink: 0;
  padding: 8px 16px;
  font: inherit;
  font-size: 13px;
  font-weight: 500;
  color: var(--ink-soft);
  background: var(--surface);
  border: 1px solid var(--line);
  border-radius: 999px;
  cursor: pointer;
  box-shadow: 0 2px 6px rgba(37, 16, 63, 0.05);
  transition: all 0.15s ease;
}

.subject-chip.active {
  color: #fff;
  background: var(--violet);
  border-color: var(--violet);
  font-weight: 600;
  box-shadow: 0 8px 16px -8px rgba(117, 70, 168, 0.8);
}

.subject-chip:focus-visible {
  outline: 3px solid rgba(117, 70, 168, 0.4);
  outline-offset: 2px;
}

/* Stats bar */
.stats-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 6px 4px 14px;
  font-size: 13px;
  color: var(--ink-soft);
}

.stats-bar strong {
  color: var(--violet-deep);
}

.sort-label {
  font-weight: 500;
}

/* Quiz Card */
.panel {
  background: var(--surface);
  border-radius: var(--radius-lg);
  padding: 18px;
  margin-bottom: 14px;
  box-shadow: 0 1px 2px rgba(37, 16, 63, 0.05), 0 12px 30px -14px rgba(37, 16, 63, 0.2);
}

.quiz-card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.subject-badge {
  font-size: 12px;
  font-weight: 600;
  color: var(--violet-deep);
  background: var(--violet-tint);
  padding: 4px 10px;
  border-radius: 8px;
}

.date-badge {
  font-size: 12px;
  color: var(--ink-soft);
}

.quiz-title {
  margin: 0 0 6px;
  font-size: 16px;
  font-weight: 600;
  color: var(--ink);
}

.quiz-desc {
  margin: 0 0 14px;
  font-size: 13px;
  line-height: 1.5;
  color: var(--ink-soft);
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.quiz-meta-info {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-bottom: 14px;
  font-size: 12px;
  font-weight: 500;
  color: var(--ink-soft);
  background: #f8f4ff;
  padding: 8px 12px;
  border-radius: 10px;
}

/* Button Row Inside Card */
.btn-row {
  display: flex;
  gap: 10px;
}

.btn-row ion-button {
  flex: 1;
}

ion-button {
  --border-radius: 14px;
  min-height: 42px;
  margin: 0;
  font-weight: 600;
  font-size: 13.5px;
  text-transform: none;
}

.btn-main {
  --background: var(--night);
  --background-activated: #3a1d63;
  --color: #fff;
  --box-shadow: 0 10px 20px -10px rgba(37, 16, 63, 0.7);
}

.btn-ghost {
  --color: var(--violet-deep);
  --background: var(--violet-tint);
  --background-activated: #e4d8fb;
}

/* Empty State */
.empty-state {
  text-align: center;
  padding: 40px 20px;
}

.ph-icon {
  width: 56px;
  height: 56px;
  margin: 0 auto 12px;
  display: grid;
  place-items: center;
  border-radius: 18px;
  background: var(--violet-tint);
  color: var(--violet);
}

.empty-state h3 {
  margin: 0;
  font-size: 15px;
  font-weight: 600;
}

.empty-state p {
  margin: 4px 0 0;
  font-size: 13px;
  color: var(--ink-soft);
}

/* Footer */
footer {
  text-align: center;
  padding: 18px 0 0;
}

footer p {
  margin: 0;
  font-size: 13px;
  color: var(--ink-soft);
}
</style>