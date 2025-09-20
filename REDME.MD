# مستندات کامپوننت TicketCreateModal

## 📋 معرفی
کامپوننت `TicketCreateModal` یک مودال کامل برای ایجاد تیکت با قابلیت‌های پیشرفته است.

## 🚀 نحوه استفاده

### 1. Import کردن کامپوننت
```vue
<script setup>
import TicketCreateModal from './components/ticket/TicketCreateModal.vue'
</script>
```

### 2. تعریف در Template
```vue
<template>
  <div>
    <button @click="openModal = true">ایجاد تیکت</button>
    
    <TicketCreateModal 
      v-model:open="openModal"
      @submit="onTicketSubmit"
      @cancel="onTicketCancel"
    />
  </div>
</template>
```

### 3. تعریف در Script
```vue
<script setup>
import { ref } from 'vue'
import TicketCreateModal from './components/ticket/TicketCreateModal.vue'

const openModal = ref(false)

function onTicketSubmit(ticketData) {
  console.log('داده‌های تیکت:', ticketData)
  // پردازش داده‌ها
}

function onTicketCancel() {
  console.log('کاربر انصراف داد')
}
</script>
```

## 📊 ساختار داده‌های خروجی

### فرمت داده‌های emit شده:
```typescript
interface TicketData {
  title: string                    // عنوان تیکت
  building: {                      // بخش انتخاب شده
    id: number | string,           // شناسه بخش
    name: string                   // نام بخش
  }
  service: {                       // سرویس انتخاب شده
    id: number | string,           // شناسه سرویس
    name: string                   // نام سرویس
  }
  receiver: {                      // گیرنده انتخاب شده
    id: number | string,           // شناسه گیرنده
    name: string                   // نام گیرنده
  }
  priority: {                      // اولویت
    id: number,                    // شناسه اولویت (1-5)
    name: string                   // نام اولویت
  }
  type: {                          // نوع تیکت
    id: number | string,           // شناسه نوع
    name: string                   // نام نوع
  }
  contact: string                  // شماره تماس
  description: string              // توضیحات (HTML)
  checklist: string[]              // لیست چک‌ها
  files: File[]                    // فایل‌های ضمیمه
}
```

### مثال داده‌های خروجی:
```javascript
{
  title: "مشکل در سیستم",
  building: {
    id: 1,
    name: "تست 1"
  },
  service: {
    id: 2,
    name: "تست 2"
  },
  receiver: {
    id: 3,
    name: "تست 3"
  },
  priority: {
    id: 3,
    name: "متوسط"
  },
  type: {
    id: 1,
    name: "ali"
  },
  contact: "09134279848",
  description: "<p>توضیحات مشکل...</p>",
  checklist: ["بررسی اول", "تست سیستم"],
  files: [File, File]
}
```

## ⚙️ Props

| نام | نوع | پیش‌فرض | توضیحات |
|-----|-----|---------|---------|
| `open` | `boolean` | `false` | وضعیت باز/بسته بودن مودال |

## 📤 Events

| نام | پارامتر | توضیحات |
|-----|---------|---------|
| `update:open` | `boolean` | تغییر وضعیت مودال |
| `submit` | `TicketData` | ارسال تیکت |
| `cancel` | - | انصراف از ایجاد تیکت |

## 🔧 کامپوننت‌های وابسته

### 1. DynamicCombo
کامپوننت انتخاب با قابلیت جستجو و نمایش آواتار

**Props:**
- `modelValue`: مقدار انتخاب شده
- `options`: آرایه گزینه‌ها
- `placeholder`: متن راهنما
- `label`: برچسب فیلد

**Events:**
- `update:modelValue`: تغییر مقدار
- `change`: تغییر انتخاب

### 2. UploadModal
مودال آپلود فایل با قابلیت پیش‌نمایش

**Props:**
- `isOpen`: وضعیت باز/بسته
- `selectedFiles`: فایل‌های انتخاب شده
- `theme`: تم رنگی

**Events:**
- `files-passed`: ارسال فایل‌ها
- `file-error`: خطای فایل

## ✅ قوانین اعتبارسنجی

### فیلدهای اجباری:
- ✅ **عنوان تیکت**: حداقل 5 کاراکتر
- ✅ **بخش**: انتخاب اجباری
- ✅ **وضعیت**: انتخاب اجباری
- ✅ **شماره تماس**: فرمت 09123456789
- ✅ **سرویس**: انتخاب اجباری
- ✅ **گیرنده**: انتخاب اجباری
- ✅ **نوع**: انتخاب اجباری
- ✅ **توضیحات**: حداقل 30 کاراکتر

### فرمت شماره تماس:
```
09123456789  ✅ صحیح
(09)123456789 ❌ غلط
0912345678   ❌ غلط (کمتر از 11 رقم)
```

## 🎨 استایل‌ها

### متغیرهای CSS:
```css
--bg: #ffffff           /* پس‌زمینه اصلی */
--accent: #4E71FF      /* رنگ اصلی */
--surface: #f0f9ff     /* پس‌زمینه سطح */
--text: #2E073F        /* رنگ متن */
--muted: #2E073F       /* رنگ متن کمرنگ */
--border: transparent  /* حاشیه */
```

### کلاس‌های مهم:
- `.modal`: کانتینر اصلی مودال
- `.field`: کانتینر فیلد
- `.input.error`: فیلد دارای خطا
- `.required`: علامت اجباری
- `.error-message`: پیام خطا

## 🔄 مثال کامل استفاده

```vue
<template>
  <div>
    <button @click="showModal = true">ایجاد تیکت جدید</button>
    
    <TicketCreateModal 
      v-model:open="showModal"
      @submit="handleTicketSubmit"
      @cancel="handleCancel"
    />
    
    <!-- نمایش داده‌های دریافت شده -->
    <div v-if="lastTicket" class="ticket-info">
      <h3>آخرین تیکت:</h3>
      <p>عنوان: {{ lastTicket.title }}</p>
      <p>بخش: {{ lastTicket.building }}</p>
      <p>تماس: {{ lastTicket.contact }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import TicketCreateModal from './components/ticket/TicketCreateModal.vue'

const showModal = ref(false)
const lastTicket = ref(null)

function handleTicketSubmit(ticketData) {
  console.log('تیکت جدید:', ticketData)
  
  // ذخیره در localStorage
  localStorage.setItem('lastTicket', JSON.stringify(ticketData))
  
  // ذخیره در state
  lastTicket.value = ticketData
  
  // ارسال به سرور
  // await sendToServer(ticketData)
  
  showModal.value = false
}

function handleCancel() {
  console.log('کاربر انصراف داد')
  showModal.value = false
}
</script>
```

## 🚨 نکات مهم

1. **اعتبارسنجی**: تمام فیلدها قبل از ارسال اعتبارسنجی می‌شوند
2. **فونت**: از فونت پلاک استفاده می‌کند
3. **فایل‌ها**: فایل‌های ضمیمه به صورت File object ارسال می‌شوند
4. **HTML**: توضیحات به صورت HTML ذخیره می‌شود
5. **Responsive**: در موبایل به صورت کامل responsive است

## 🐛 عیب‌یابی

### مشکلات رایج:
1. **مودال باز نمی‌شود**: بررسی کنید `v-model:open` درست تنظیم شده
2. **داده‌ها دریافت نمی‌شود**: بررسی کنید event handler درست تعریف شده
3. **اعتبارسنجی کار نمی‌کند**: بررسی کنید تمام فیلدهای اجباری پر شده‌اند

### Console Logs:
```javascript
// برای دیباگ، این کد را اضافه کنید:
function handleTicketSubmit(ticketData) {
  console.log('Ticket Data:', ticketData)
  console.log('Validation passed:', true)
}
```
