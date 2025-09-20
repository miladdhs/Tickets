<script setup lang="ts">
import { ref } from 'vue'
import TicketCreateModal from './components/ticket/TicketCreateModal.vue'

interface TicketData {
  title: string
  building: { id: any; name: string }
  service: { id: any; name: string }
  receiver: { id: any; name: string }
  priority: { id: any; name: string }
  type: { id: any; name: string }
  contact: string
  description: string
  checklist: string[]
  files: File[]
}

const open = ref(false)
const showSuccess = ref(false)
const lastTicket = ref<TicketData | null>(null)

function onSubmit(ticketData: TicketData){
  console.log('📋 داده‌های تیکت دریافت شد:', ticketData)
  
  // ذخیره داده‌ها
  lastTicket.value = ticketData
  
  // نمایش پیام موفقیت
  showSuccess.value = true
  setTimeout(() => {
    showSuccess.value = false
  }, 3000)
  
  // در اینجا می‌توانید داده‌ها را به سرور ارسال کنید
  // sendToServer(ticketData)
}

function onCancel(){
  console.log('❌ کاربر انصراف داد')
}
</script>

<template>
  <div style="display:flex; flex-direction:column; align-items:center; justify-content:center; height:100vh; gap:20px; background:#e5e7eb; position:relative; padding:20px;">
    <button style="height:40px; padding:0 16px; border-radius:10px; border:1px solid #cbd5e1; background:#fff; cursor:pointer; font-family:'Pelak', sans-serif;" @click="open = true">ایجاد تیکت</button>
    
    <TicketCreateModal 
      v-model:open="open" 
      @submit="onSubmit" 
      @cancel="onCancel" 
    />
    
    <!-- نمایش داده‌های آخرین تیکت -->
    <div v-if="lastTicket" style="background:white; padding:20px; border-radius:12px; box-shadow:0 4px 12px rgba(0,0,0,0.1); max-width:500px; width:100%; font-family:'Pelak', sans-serif;">
      <h3 style="margin:0 0 15px 0; color:#2E073F;">📋 آخرین تیکت ایجاد شده:</h3>
      <div style="display:grid; gap:8px; font-size:14px;">
        <div><strong>عنوان:</strong> {{ lastTicket.title }}</div>
        <div><strong>بخش:</strong> {{ lastTicket.building?.name || 'نامشخص' }}</div>
        <div><strong>سرویس:</strong> {{ lastTicket.service?.name || 'نامشخص' }}</div>
        <div><strong>گیرنده:</strong> {{ lastTicket.receiver?.name || 'نامشخص' }}</div>
        <div><strong>اولویت:</strong> {{ lastTicket.priority?.name || 'نامشخص' }}</div>
        <div><strong>نوع:</strong> {{ lastTicket.type?.name || 'نامشخص' }}</div>
        <div><strong>تماس:</strong> {{ lastTicket.contact }}</div>
        <div><strong>چک‌لیست:</strong> {{ lastTicket.checklist?.length || 0 }} آیتم</div>
        <div><strong>فایل‌ها:</strong> {{ lastTicket.files?.length || 0 }} فایل</div>
      </div>
    </div>
    
    <!-- Success Message -->
    <div v-if="showSuccess" style="position:fixed; top:20px; right:20px; background:#10b981; color:white; padding:12px 20px; border-radius:8px; font-family:'Pelak', sans-serif; font-weight:500; box-shadow:0 4px 12px rgba(16, 185, 129, 0.3); z-index:1000;">
      ✅ تیکت با موفقیت ایجاد شد و فایل JSON دانلود شد!
    </div>
  </div>
</template>

<style>
@font-face {
  font-family: 'IRANYekanXFaNum';
  src: url('/src/font/IRANYekanX Pro/Farsi numerals/Webfonts/Woff2/IRANYekanXFaNum-Regular.woff2') format('woff2');
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}
@font-face {
  font-family: 'IRANYekanXFaNum';
  src: url('/src/font/IRANYekanX Pro/Farsi numerals/Webfonts/Woff2/IRANYekanXFaNum-Bold.woff2') format('woff2');
  font-weight: 700;
  font-style: normal;
  font-display: swap;
}
@font-face {
  font-family: 'Kalameh';
  src: url('/src/font/Kalameh-Medium.ttf') format('truetype');
  font-weight: 500;
  font-style: normal;
  font-display: swap;
}
@font-face {
  font-family: 'Pelak';
  src: url('/src/font/Pelak/Pelak-Thin.ttf') format('truetype');
  font-weight: 100;
  font-style: normal;
  font-display: swap;
}
@font-face {
  font-family: 'Pelak';
  src: url('/src/font/Pelak/Pelak-light.ttf') format('truetype');
  font-weight: 300;
  font-style: normal;
  font-display: swap;
}
@font-face {
  font-family: 'Pelak';
  src: url('/src/font/Pelak/Pelak-Regular.ttf') format('truetype');
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}
@font-face {
  font-family: 'Pelak';
  src: url('/src/font/Pelak/Pelak-Medium.ttf') format('truetype');
  font-weight: 500;
  font-style: normal;
  font-display: swap;
}
@font-face {
  font-family: 'Pelak';
  src: url('/src/font/Pelak/Pelak-SemiBold.ttf') format('truetype');
  font-weight: 600;
  font-style: normal;
  font-display: swap;
}
@font-face {
  font-family: 'Pelak';
  src: url('/src/font/Pelak/Pelak-Bold.ttf') format('truetype');
  font-weight: 700;
  font-style: normal;
  font-display: swap;
}
@font-face {
  font-family: 'Pelak';
  src: url('/src/font/Pelak/Pelak-ExtraBold.ttf') format('truetype');
  font-weight: 800;
  font-style: normal;
  font-display: swap;
}
@font-face {
  font-family: 'Pelak';
  src: url('/src/font/Pelak/Pelak-Black.ttf') format('truetype');
  font-weight: 900;
  font-style: normal;
  font-display: swap;
}
html, body { margin:0; font-family: 'Pelak', IRANYekanXFaNum, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial, "Apple Color Emoji", "Segoe UI Emoji"; -webkit-font-smoothing: antialiased; -moz-osx-font-smoothing: grayscale }
</style>


