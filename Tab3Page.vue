<template>
  <ion-page>
    <ion-header>
      <ion-toolbar class="main-toolbar">
        <ion-title>✨ AI SMART QUIZ</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="login-page">

      <!-- HERO -->
      <div class="hero">
        <div class="hero-icon">🧠</div>
        <h1>ยินดีต้อนรับกลับมา</h1>
        <p>เข้าสู่ระบบเพื่อสร้างข้อสอบอัจฉริยะจากเนื้อหาการเรียนของคุณ</p>
        <span class="hero-tag">✨ Powered by Artificial Intelligence</span>
      </div>

      <div class="content-wrapper">

        <!-- LOGIN CARD -->
        <ion-card class="app-card">
          <ion-card-header>
            <ion-card-title>🔐 เข้าสู่ระบบ</ion-card-title>
            <p>กรอกอีเมลและรหัสผ่านเพื่อเริ่มใช้งาน</p>
          </ion-card-header>

          <ion-card-content>

            <label class="field-label" for="email">📧 อีเมล</label>
            <input
              id="email"
              v-model="email"
              type="email"
              class="custom-input"
              placeholder="example@email.com"
              autocomplete="email"
              :disabled="loading"
              @keyup.enter="login"
            />

            <label class="field-label" for="password">🔑 รหัสผ่าน</label>
            <div class="password-box">
              <input
                id="password"
                v-model="password"
                :type="showPassword ? 'text' : 'password'"
                class="custom-input"
                placeholder="กรอกรหัสผ่านของคุณ"
                autocomplete="current-password"
                :disabled="loading"
                @keyup.enter="login"
              />
              <button
                type="button"
                class="toggle-eye"
                :aria-label="showPassword ? 'ซ่อนรหัสผ่าน' : 'แสดงรหัสผ่าน'"
                @click="showPassword = !showPassword"
              >
                {{ showPassword ? '🙈' : '👁️' }}
              </button>
            </div>

            <div class="options-row">
              <label class="remember">
                <input type="checkbox" v-model="remember" />
                <span>จดจำฉันไว้</span>
              </label>

              <a href="#" class="link" @click.prevent="forgotPassword">
                ลืมรหัสผ่าน?
              </a>
            </div>

            <p v-if="message" class="status" :class="{ error: isError }">
              {{ message }}
            </p>

            <ion-button
              expand="block"
              class="login-button"
              @click="login"
              :disabled="loading"
            >
              <span v-if="!loading">🚀 เข้าสู่ระบบ</span>
              <span v-else>
                <ion-spinner name="crescent" class="btn-spinner"></ion-spinner>
                กำลังเข้าสู่ระบบ...
              </span>
            </ion-button>

            <div class="setting-note">
              💡 ยังไม่มีบัญชี?
              <a href="#" class="link" @click.prevent="goRegister">สมัครสมาชิกฟรี</a>
            </div>

          </ion-card-content>
        </ion-card>

        <!-- FEATURE CARD -->
        <ion-card class="generate-card">
          <ion-card-content>
            <div class="generate-icon">📚</div>
            <h2>ทำไมต้อง AI Smart Quiz?</h2>
            <p>ถ่ายภาพเนื้อหา แล้วให้ AI สรุปและสร้างข้อสอบให้ภายในไม่กี่วินาที</p>
          </ion-card-content>
        </ion-card>

      </div>

      <footer>
        <p>Nexgen</p>
        <small>AI Smart Quiz Application</small>
      </footer>

    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';

import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardContent,
  IonButton,
  IonSpinner
} from '@ionic/vue';

const router = useRouter();

// Backend URL (เหมือนหน้าสร้างข้อสอบ)
const API_URL = 'http://localhost:3000';

// ================================
// STATE
// ================================

const email = ref('');
const password = ref('');
const remember = ref(false);
const showPassword = ref(false);

const loading = ref(false);
const message = ref('');
const isError = ref(false);

// ================================
// HELPERS
// ================================

function showMessage(text: string, error = false) {
  message.value = text;
  isError.value = error;
}

function isValidEmail(value: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);
}

// ================================
// LOGIN
// ================================

async function login() {
  if (!email.value.trim() || !password.value) {
    showMessage('กรุณากรอกอีเมลและรหัสผ่านให้ครบ', true);
    return;
  }

  if (!isValidEmail(email.value.trim())) {
    showMessage('รูปแบบอีเมลไม่ถูกต้อง', true);
    return;
  }

  try {
    loading.value = true;
    showMessage('');

    const response = await fetch(`${API_URL}/api/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        email: email.value.trim(),
        password: password.value
      })
    });

    const data = await response.json();

    if (!response.ok) {
      throw new Error(data.error || 'อีเมลหรือรหัสผ่านไม่ถูกต้อง');
    }

    if (data.token) {
      localStorage.setItem('token', data.token);
    }

    if (remember.value) {
      localStorage.setItem('rememberedEmail', email.value.trim());
    } else {
      localStorage.removeItem('rememberedEmail');
    }

    showMessage('เข้าสู่ระบบสำเร็จแล้ว! 🎉');

    // เปลี่ยนเส้นทางไปหน้าสร้างข้อสอบ (แก้ path ให้ตรงกับ router ของคุณ)
    router.push('/quiz');

  } catch (error: any) {
    console.error(error);
    showMessage(error.message || 'ไม่สามารถเชื่อมต่อเซิร์ฟเวอร์ได้', true);
  } finally {
    loading.value = false;
  }
}

// ================================
// OTHER ACTIONS
// ================================

function forgotPassword() {
  showMessage('ฟีเจอร์รีเซ็ตรหัสผ่านจะเปิดให้ใช้งานเร็ว ๆ นี้');
}

function goRegister() {
  router.push('/register');
}

// ================================
// INIT
// ================================

onMounted(() => {
  const saved = localStorage.getItem('rememberedEmail');
  if (saved) {
    email.value = saved;
    remember.value = true;
  }
});
</script>

<style scoped>
.login-page {
  --background: #f6f2ff;
}

/* HEADER */
.main-toolbar {
  --background: #7546a8;
  --color: white;
}

/* HERO */
.hero {
  background: linear-gradient(135deg, #7162d9, #934eb0);
  color: white;
  text-align: center;
  padding: 35px 15px 40px;
  border-radius: 0 0 35px 35px;
}

.hero-icon {
  font-size: 55px;
  animation: float 2s infinite ease-in-out;
}

.hero h1 {
  font-size: 28px;
  font-weight: 800;
  margin: 10px 0;
}

.hero p {
  font-size: 14px;
  line-height: 1.6;
}

.hero-tag {
  display: inline-block;
  background: rgba(255, 255, 255, 0.2);
  padding: 8px 15px;
  border-radius: 30px;
  font-size: 12px;
  margin-top: 10px;
}

/* CONTENT */
.content-wrapper {
  max-width: 500px;
  margin: auto;
  padding: 10px;
}

/* CARDS */
.app-card {
  border-radius: 22px;
  margin: 18px 0;
  padding: 5px;
  box-shadow: 0 5px 20px rgba(100, 60, 150, 0.09);
}

ion-card-title {
  color: #7546a8;
  font-size: 18px;
  font-weight: bold;
}

ion-card-header p {
  color: #888;
  font-size: 13px;
}

/* FORM */
.field-label {
  display: block;
  color: #7546a8;
  font-weight: bold;
  margin: 18px 0 8px;
}

.custom-input {
  width: 100%;
  padding: 14px;
  border: 1px solid #e3d6f2;
  border-radius: 12px;
  background: #faf8ff;
  color: #514260;
  font-size: 15px;
  outline: none;
  box-sizing: border-box;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.custom-input:focus {
  border-color: #7546a8;
  box-shadow: 0 0 0 3px rgba(117, 70, 168, 0.15);
}

.custom-input::placeholder {
  color: #b9aecb;
}

.password-box {
  position: relative;
}

.password-box .custom-input {
  padding-right: 52px;
}

.toggle-eye {
  position: absolute;
  top: 50%;
  right: 8px;
  transform: translateY(-50%);
  background: none;
  border: none;
  font-size: 20px;
  padding: 6px 8px;
  cursor: pointer;
  border-radius: 10px;
}

.toggle-eye:focus-visible {
  outline: 2px solid #7546a8;
}

.options-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 16px;
  font-size: 13px;
}

.remember {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #514260;
  cursor: pointer;
}

.remember input {
  accent-color: #7546a8;
  width: 16px;
  height: 16px;
}

.link {
  color: #7546a8;
  font-weight: bold;
  text-decoration: none;
}

.link:hover {
  text-decoration: underline;
}

.status {
  color: #7546a8;
  text-align: center;
  font-weight: bold;
  margin: 15px 0 0;
  font-size: 14px;
}

.status.error {
  color: #d33c5a;
}

ion-button {
  margin-top: 12px;
  --border-radius: 14px;
  font-weight: bold;
}

.login-button {
  --background: linear-gradient(135deg, #7546a8, #a15ab8);
  min-height: 52px;
  margin-top: 20px;
}

.btn-spinner {
  width: 18px;
  height: 18px;
  vertical-align: middle;
  margin-right: 6px;
}

.setting-note {
  background: #f1eaff;
  padding: 14px;
  border-radius: 12px;
  color: #7546a8;
  margin-top: 20px;
  font-size: 13px;
  line-height: 1.6;
  text-align: center;
}

/* FEATURE CARD */
.generate-card {
  background: linear-gradient(135deg, #7546a8, #a15ab8);
  border-radius: 24px;
  text-align: center;
  color: white;
  margin: 20px 0;
  padding: 15px;
}

.generate-icon {
  font-size: 50px;
}

.generate-card h2 {
  font-size: 20px;
  font-weight: bold;
  color: white;
}

.generate-card p {
  color: #f2eaff;
  font-size: 14px;
}

/* FOOTER */
footer {
  text-align: center;
  padding: 20px;
  color: #7546a8;
}

footer p {
  font-weight: bold;
}

footer small {
  color: #999;
}

/* ANIMATION */
@keyframes float {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-9px);
  }
}

@media (prefers-reduced-motion: reduce) {
  .hero-icon {
    animation: none;
  }
}
</style>