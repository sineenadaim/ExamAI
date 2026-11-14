
<template>
  <ion-page>
    <ion-header>
      <ion-toolbar class="main-toolbar">
        <ion-title>✨ AI SMART QUIZ</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="quiz-page">

      <!-- HERO -->
      <div class="hero">
        <div class="hero-icon">🧠</div>
        <h1>AI Smart Quiz</h1>
        <p>เปลี่ยนเนื้อหาการเรียน ให้กลายเป็นข้อสอบอัจฉริยะ</p>
        <span class="hero-tag">✨ Powered by Artificial Intelligence</span>
      </div>

      <div class="content-wrapper">

        <!-- STEP 1: UPLOAD -->
        <ion-card class="app-card">
          <ion-card-header>
            <ion-card-title>📚 ขั้นตอนที่ 1 : เพิ่มเนื้อหาการเรียน</ion-card-title>
            <p>ถ่ายภาพหรือเลือกรูปภาพเนื้อหาที่ต้องการสร้างข้อสอบ</p>
          </ion-card-header>

          <ion-card-content>

            <div class="camera-box">
              <video
                ref="video"
                autoplay
                playsinline
                muted
                v-show="cameraOn"
              ></video>

              <div v-if="!cameraOn && !photoUrl" class="camera-placeholder">
                <div class="big-icon">📸</div>
                <h3>AI Quiz Scanner</h3>
                <p>พร้อมสแกนเนื้อหาการเรียนของคุณ</p>
              </div>
            </div>

            <p class="status">{{ message }}</p>

            <ion-button
              expand="block"
              @click="startCamera"
              :disabled="cameraOn || loading"
            >
              📷 เปิดกล้องโน้ตบุ๊ก
            </ion-button>

            <ion-button
              expand="block"
              color="success"
              @click="capturePhoto"
              :disabled="!cameraOn || loading"
            >
              📸 ถ่ายภาพเนื้อหา
            </ion-button>

            <ion-button
              expand="block"
              color="medium"
              fill="outline"
              @click="stopCamera"
              :disabled="!cameraOn"
            >
              ปิดกล้อง
            </ion-button>

            <div class="or-text">หรือ</div>

            <input
              ref="fileInput"
              type="file"
              accept="image/*"
              hidden
              @change="onFileSelected"
            />

            <ion-button
              expand="block"
              fill="outline"
              @click="openFilePicker"
              :disabled="loading"
            >
              🖼️ เลือกรูปภาพจากเครื่อง
            </ion-button>

            <!-- IMAGE PREVIEW -->
            <div v-if="photoUrl" class="preview">
              <h3>✅ ภาพเนื้อหาของคุณ</h3>
              <img :src="photoUrl" alt="ภาพเนื้อหาการเรียน" />
              <p>{{ selectedImage?.name }}</p>

              <ion-button
                expand="block"
                color="danger"
                fill="outline"
                @click="deletePhoto"
                :disabled="loading"
              >
                🗑️ ลบภาพและเลือกใหม่
              </ion-button>
            </div>

          </ion-card-content>
        </ion-card>

        <!-- STEP 2: SETTINGS -->
        <ion-card class="app-card">
          <ion-card-header>
            <ion-card-title>⚙️ ขั้นตอนที่ 2 : ตั้งค่าข้อสอบ</ion-card-title>
            <p>กำหนดรูปแบบข้อสอบตามต้องการ</p>
          </ion-card-header>

          <ion-card-content>

            <label class="field-label">📝 จำนวนข้อสอบ</label>
            <select v-model="questionCount" class="custom-select">
              <option value="5">5 ข้อ</option>
              <option value="10">10 ข้อ</option>
              <option value="15">15 ข้อ</option>
              <option value="20">20 ข้อ</option>
            </select>

            <label class="field-label">🎯 ระดับความยาก</label>
            <select v-model="difficulty" class="custom-select">
              <option value="ง่าย">ง่าย</option>
              <option value="ปานกลาง">ปานกลาง</option>
              <option value="ยาก">ยาก</option>
              <option value="ผสม">ผสมทุกระดับ</option>
            </select>

            <label class="field-label">📋 รูปแบบข้อสอบ</label>
            <select v-model="quizType" class="custom-select">
              <option value="ปรนัย 4 ตัวเลือก">ปรนัย 4 ตัวเลือก</option>
              <option value="อัตนัย">อัตนัย</option>
              <option value="ผสม">ผสมปรนัยและอัตนัย</option>
            </select>

            <div class="setting-note">
              💡 AI จะวิเคราะห์เนื้อหาจากภาพและสร้างข้อสอบตามการตั้งค่าของคุณ
            </div>

          </ion-card-content>
        </ion-card>

        <!-- STEP 3: GENERATE -->
        <ion-card class="generate-card">
          <ion-card-content>
            <div class="generate-icon">✨</div>
            <h2>พร้อมสร้างข้อสอบหรือยัง?</h2>
            <p>ให้ AI ช่วยสรุปและสร้างข้อสอบจากเนื้อหาของคุณ</p>

            <ion-button
              expand="block"
              class="generate-button"
              @click="generateQuiz"
              :disabled="!selectedImage || loading"
            >
              <span v-if="!loading">🚀 สร้างข้อสอบด้วย AI</span>
              <span v-else>⏳ AI กำลังประมวลผล...</span>
            </ion-button>

            <p class="small-note">
              กรุณาเลือกรูปภาพก่อนเริ่มสร้างข้อสอบ
            </p>
          </ion-card-content>
        </ion-card>

        <!-- LOADING -->
        <div v-if="loading" class="loading-box">
          <div class="loading-icon">🤖</div>
          <h3>AI กำลังอ่านเนื้อหาของคุณ</h3>
          <p>กำลังสรุปเนื้อหาและสร้างข้อสอบ กรุณารอสักครู่</p>
          <ion-spinner name="crescent" color="primary"></ion-spinner>
        </div>

        <!-- RESULTS -->
        <ion-card v-if="result" class="result-card">
          <ion-card-header>
            <ion-card-title>🎉 ผลลัพธ์จาก AI</ion-card-title>
            <p>สรุปเนื้อหาและข้อสอบที่สร้างจากภาพของคุณ</p>
          </ion-card-header>

          <ion-card-content>
            <div class="result-content">{{ result }}</div>

            <ion-button
              expand="block"
              fill="outline"
              @click="copyResult"
            >
              📋 คัดลอกผลลัพธ์
            </ion-button>


            <ion-button
              expand="block"
              color="danger"
              @click="downloadPDF"
            >
               📄 ดาวน์โหลดข้อสอบ PDF
            </ion-button>

            <ion-button
              expand="block"
              color="medium"
              fill="outline"
              @click="clearResult"
            >
              🔄 ล้างผลลัพธ์
            </ion-button>
          </ion-card-content>
        </ion-card>

      </div>

      <footer>
        <p>Made with 💜 by Praewa</p>
        <small>AI Smart Quiz Application</small>
      </footer>

    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onBeforeUnmount } from 'vue';

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


// ================================
// DOWNLOAD PDF
// ================================

function escapeHtml(text: string): string {
  return text
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#039;');
}

function downloadPDF() {
  if (!result.value.trim()) {
    message.value = 'ไม่มีข้อสอบสำหรับสร้าง PDF';
    return;
  }

  // เปิดหน้าต่างสำหรับพิมพ์ PDF
  const printWindow = window.open('', '_blank');

  if (!printWindow) {
    message.value = 'กรุณาอนุญาตให้เว็บไซต์เปิดหน้าต่างใหม่';
    return;
  }

  const safeResult = escapeHtml(result.value);

  const html = `
    <!DOCTYPE html>
    <html lang="th">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">

      <title>AI Smart Quiz - ข้อสอบ</title>

      <style>
        @page {
          size: A4;
          margin: 20mm;
        }

        * {
          box-sizing: border-box;
        }

        body {
          font-family: "Noto Sans Thai", Tahoma, Arial, sans-serif;
          color: #333;
          background: white;
          line-height: 1.8;
          font-size: 14px;
        }

        h1 {
          text-align: center;
          color: #7546a8;
          font-size: 24px;
          margin-bottom: 5px;
        }

        .subtitle {
          text-align: center;
          color: #888;
          margin-bottom: 25px;
          font-size: 12px;
        }

        .divider {
          border-top: 2px solid #7546a8;
          margin-bottom: 25px;
        }

        .content {
          white-space: pre-wrap;
          overflow-wrap: anywhere;
          font-size: 14px;
          line-height: 1.9;
        }

        .footer {
          margin-top: 40px;
          padding-top: 10px;
          border-top: 1px solid #ddd;
          text-align: center;
          color: #999;
          font-size: 11px;
        }

        @media print {
          body {
            -webkit-print-color-adjust: exact;
            print-color-adjust: exact;
          }
        }
      </style>
    </head>

    <body>

      <h1>AI SMART QUIZ</h1>

      <div class="subtitle">
        แบบทดสอบที่สร้างด้วยปัญญาประดิษฐ์
      </div>

      <div class="divider"></div>

      <div class="content">${safeResult}</div>

      <div class="footer">
        Generated by AI Smart Quiz
      </div>

      <script>
        window.onload = function() {
          setTimeout(function() {
            window.print();
          }, 500);
        };
      <\/script>

    </body>
    </html>
  `;

  printWindow.document.open();
  printWindow.document.write(html);
  printWindow.document.close();

  message.value = 'เตรียมเอกสาร PDF เรียบร้อยแล้ว';
}


// ================================
// CAMERA
// ================================

const video = ref<HTMLVideoElement | null>(null);
const fileInput = ref<HTMLInputElement | null>(null);

const cameraOn = ref(false);
const message = ref('พร้อมเปิดกล้อง');

let stream: MediaStream | null = null;

// ================================
// IMAGE
// ================================

const photoUrl = ref<string | null>(null);
const selectedImage = ref<File | null>(null);

// ================================
// QUIZ SETTINGS
// ================================

const questionCount = ref('10');
const difficulty = ref('ปานกลาง');
const quizType = ref('ปรนัย 4 ตัวเลือก');

// ================================
// AI RESULT
// ================================

const loading = ref(false);
const result = ref('');

// Backend URL
const API_URL = 'http://localhost:3000';

// ================================
// OPEN CAMERA
// ================================

async function startCamera() {
  try {
    if (!navigator.mediaDevices?.getUserMedia) {
      throw new Error('browser');
    }

    message.value = 'กำลังเปิดกล้อง...';

    stream = await navigator.mediaDevices.getUserMedia({
      video: {
        facingMode: 'user',
        width: { ideal: 1280 },
        height: { ideal: 720 }
      },
      audio: false
    });

    if (!video.value) {
      throw new Error('ไม่พบพื้นที่แสดงกล้อง');
    }

    video.value.srcObject = stream;
    await video.value.play();

    cameraOn.value = true;
    message.value = '🟢 กล้องพร้อมใช้งานแล้ว';

  } catch (error: any) {
    console.error(error);

    if (error.name === 'NotAllowedError') {
      message.value = 'กรุณาอนุญาตให้เว็บไซต์ใช้กล้อง';
    } else if (error.name === 'NotFoundError') {
      message.value = 'ไม่พบกล้องในโน้ตบุ๊ก';
    } else {
      message.value = 'ไม่สามารถเปิดกล้องได้ กรุณาตรวจสอบอีกครั้ง';
    }
  }
}

// ================================
// CAPTURE PHOTO
// ================================

function capturePhoto() {
  if (!video.value || !cameraOn.value) return;

  const canvas = document.createElement('canvas');

  canvas.width = video.value.videoWidth;
  canvas.height = video.value.videoHeight;

  const context = canvas.getContext('2d');

  if (!context) return;

  context.drawImage(
    video.value,
    0,
    0,
    canvas.width,
    canvas.height
  );

  canvas.toBlob((blob) => {
    if (!blob) {
      message.value = 'ไม่สามารถบันทึกภาพได้';
      return;
    }

    const file = new File(
      [blob],
      'camera-capture.jpg',
      { type: 'image/jpeg' }
    );

    setSelectedImage(file);

    message.value = 'ถ่ายภาพสำเร็จแล้ว! 🎉';

  }, 'image/jpeg', 0.9);
}

// ================================
// UPLOAD IMAGE
// ================================

function openFilePicker() {
  fileInput.value?.click();
}

function onFileSelected(event: Event) {
  const target = event.target as HTMLInputElement;
  const file = target.files?.[0];

  if (!file) return;

  if (!file.type.startsWith('image/')) {
    message.value = 'กรุณาเลือกไฟล์รูปภาพเท่านั้น';
    target.value = '';
    return;
  }

  setSelectedImage(file);

  message.value = 'เลือกภาพเรียบร้อยแล้ว! 🎉';

  target.value = '';
}

// ================================
// SET IMAGE
// ================================

function setSelectedImage(file: File) {
  if (photoUrl.value) {
    URL.revokeObjectURL(photoUrl.value);
  }

  selectedImage.value = file;
  photoUrl.value = URL.createObjectURL(file);

  result.value = '';
}

// ================================
// STOP CAMERA
// ================================

function stopCamera() {
  if (stream) {
    stream.getTracks().forEach(track => track.stop());
    stream = null;
  }

  if (video.value) {
    video.value.srcObject = null;
  }

  cameraOn.value = false;
  message.value = 'ปิดกล้องแล้ว';
}

// ================================
// DELETE IMAGE
// ================================

function deletePhoto() {
  if (photoUrl.value) {
    URL.revokeObjectURL(photoUrl.value);
  }

  photoUrl.value = null;
  selectedImage.value = null;
  result.value = '';

  message.value = 'ลบภาพเรียบร้อยแล้ว';
}

// ================================
// GENERATE AI QUIZ
// ================================

async function generateQuiz() {
  if (!selectedImage.value) {
    message.value = 'กรุณาเลือกรูปภาพก่อน';
    return;
  }

  try {
    loading.value = true;
    result.value = '';

    const formData = new FormData();

    formData.append('image', selectedImage.value);
    formData.append('questionCount', questionCount.value);
    formData.append('difficulty', difficulty.value);
    formData.append('quizType', quizType.value);

    const response = await fetch(
      `${API_URL}/api/generate-quiz`,
      {
        method: 'POST',
        body: formData
      }
    );

    const data = await response.json();

    if (!response.ok) {
      throw new Error(data.error || 'ไม่สามารถสร้างข้อสอบได้');
    }

    result.value = data.result || 'AI ไม่ได้ส่งผลลัพธ์กลับมา';

    message.value = 'สร้างข้อสอบสำเร็จแล้ว! 🎉';

  } catch (error: any) {
    console.error(error);

    message.value = 'เกิดข้อผิดพลาดในการเชื่อมต่อ AI';

    result.value =
      'ไม่สามารถสร้างข้อสอบได้: ' +
      (error.message || 'กรุณาตรวจสอบ Backend และ API Key');

  } finally {
    loading.value = false;
  }
}

// ================================
// COPY RESULT
// ================================

async function copyResult() {
  try {
    await navigator.clipboard.writeText(result.value);
    message.value = 'คัดลอกผลลัพธ์แล้ว! 📋';
  } catch {
    message.value = 'ไม่สามารถคัดลอกได้';
  }
}

// ================================
// CLEAR RESULT
// ================================

function clearResult() {
  result.value = '';
  message.value = 'ล้างผลลัพธ์เรียบร้อยแล้ว';
}

// ================================
// CLEAN UP
// ================================

onBeforeUnmount(() => {
  stopCamera();

  if (photoUrl.value) {
    URL.revokeObjectURL(photoUrl.value);
  }
});
</script>

<style scoped>
.quiz-page {
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
  font-size: 30px;
  font-weight: 800;
  margin: 10px 0;
}

.hero p {
  font-size: 14px;
  line-height: 1.6;
}

.hero-tag {
  display: inline-block;
  background: rgba(255,255,255,0.2);
  padding: 8px 15px;
  border-radius: 30px;
  font-size: 12px;
  margin-top: 10px;
}

/* CONTENT */
.content-wrapper {
  max-width: 650px;
  margin: auto;
  padding: 10px;
}

/* CARDS */
.app-card {
  border-radius: 22px;
  margin: 18px 0;
  padding: 5px;
  box-shadow: 0 5px 20px rgba(100,60,150,0.09);
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

/* CAMERA */
.camera-box {
  width: 100%;
  min-height: 230px;
  background: #21182e;
  border-radius: 17px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

.camera-box video {
  width: 100%;
  display: block;
}

.camera-placeholder {
  text-align: center;
  color: white;
  padding: 25px;
}

.big-icon {
  font-size: 65px;
}

.camera-placeholder h3 {
  color: white;
  font-weight: bold;
}

.camera-placeholder p {
  color: #ddd;
}

.status {
  color: #7546a8;
  text-align: center;
  font-weight: bold;
  margin: 15px 0;
}

ion-button {
  margin-top: 12px;
  --border-radius: 14px;
  font-weight: bold;
}

/* IMAGE */
.preview {
  margin-top: 22px;
  text-align: center;
}

.preview h3 {
  color: #7546a8;
}

.preview img {
  width: 100%;
  border-radius: 15px;
  margin-top: 10px;
}

.preview p {
  color: #777;
  font-size: 13px;
  overflow-wrap: anywhere;
}

.or-text {
  text-align: center;
  color: #999;
  margin: 15px;
}

/* SETTINGS */
.field-label {
  display: block;
  color: #7546a8;
  font-weight: bold;
  margin: 18px 0 8px;
}

.custom-select {
  width: 100%;
  padding: 14px;
  border: 1px solid #e3d6f2;
  border-radius: 12px;
  background: #faf8ff;
  color: #514260;
  font-size: 15px;
  outline: none;
}

.setting-note {
  background: #f1eaff;
  padding: 14px;
  border-radius: 12px;
  color: #7546a8;
  margin-top: 20px;
  font-size: 13px;
  line-height: 1.6;
}

/* GENERATE */
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
  font-size: 22px;
  font-weight: bold;
  color: white;
}

.generate-card p {
  color: #f2eaff;
  font-size: 14px;
}

.generate-button {
  --background: white;
  --color: #7546a8;
  --border-radius: 15px;
  font-weight: bold;
  min-height: 52px;
}

.small-note {
  font-size: 12px !important;
}

/* LOADING */
.loading-box {
  background: white;
  border-radius: 20px;
  padding: 25px;
  text-align: center;
  margin: 20px 0;
}

.loading-icon {
  font-size: 50px;
  animation: float 1.5s infinite ease-in-out;
}

.loading-box h3 {
  color: #7546a8;
}

.loading-box p {
  color: #888;
  font-size: 13px;
}

ion-spinner {
  margin: 15px;
  width: 35px;
  height: 35px;
}

/* RESULTS */
.result-card {
  border-radius: 22px;
  margin: 20px 0;
}

.result-content {
  background: #f8f4ff;
  color: #40334e;
  border-radius: 15px;
  padding: 18px;
  white-space: pre-wrap;
  overflow-wrap: anywhere;
  line-height: 1.8;
  font-size: 14px;
  margin-bottom: 15px;
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
</style>