<script setup lang="ts">
import { ref, reactive, computed, watch, onMounted, onBeforeUnmount } from 'vue';
import UploadModal from './UploadModal.vue';

type ComboItem = { label: string; value: string | number; color?: string; avatarUrl?: string; meta?: string };
type Option = string | number | { label: string; value: string | number; color?: string; avatarUrl?: string; meta?: string };

const props = withDefaults(defineProps<{
  open: boolean;
}>(), { open: false });

const emit = defineEmits<{
  (e: 'update:open', v: boolean): void;
  (e: 'submit', payload: any): void;
  (e: 'cancel'): void;
}>();

const isOpen = ref<boolean>(props.open);
const dialogRef = ref<HTMLDialogElement | null>(null);

watch(() => props.open, v => {
  isOpen.value = !!v;
  if (dialogRef.value) {
    if (v) {
      dialogRef.value.showModal();
    } else {
      dialogRef.value.close();
    }
  }
});

watch(isOpen, v => emit('update:open', v));

function handleDialogClose() {
  isOpen.value = false;
}

// Dynamic Combo Helper Functions
function getLabel(opt: Option, itemLabel: string = 'label'): string {
  if (typeof opt === 'string' || typeof opt === 'number') return String(opt);
  return String((opt as any)[itemLabel] ?? '');
}

function getValue(opt: Option, itemValue: string = 'value'): string | number {
  if (typeof opt === 'string' || typeof opt === 'number') return opt;
  return (opt as any)[itemValue] as any;
}

function getAvatar(opt: Option): string | undefined {
  if (typeof opt === 'string' || typeof opt === 'number') return undefined;
  return (opt as any).avatarUrl;
}

// Real data for combos
const buildingItems = computed<ComboItem[]>(() => [
  { label: 'اداری', value: 1 },
  { label: 'فنی', value: 2 },
  { label: 'مالی', value: 3 },
  { label: 'منابع انسانی', value: 4 },
  { label: 'فروش', value: 5 },
  { label: 'بازاریابی', value: 6 },
  { label: 'پشتیبانی', value: 7 },
  { label: 'توسعه', value: 8 },
  { label: 'کیفیت', value: 9 },
  { label: 'عملیات', value: 10 }
]);

const serviceItems = computed<ComboItem[]>(() => [
  { label: 'تعمیر و نگهداری', value: 1 },
  { label: 'نصب و راه‌اندازی', value: 2 },
  { label: 'پشتیبانی فنی', value: 3 },
  { label: 'آموزش', value: 4 },
  { label: 'مشاوره', value: 5 },
  { label: 'توسعه نرم‌افزار', value: 6 },
  { label: 'پیکربندی سیستم', value: 7 },
  { label: 'بک‌آپ و بازیابی', value: 8 }
]);

const receiverItems = computed<any[]>(() => [
  { label: 'احمد محمدی', value: 1, avatarUrl: 'https://i.pravatar.cc/64?img=1', meta: 'مدیر فنی', isOnline: true },
  { label: 'فاطمه احمدی', value: 2, avatarUrl: 'https://i.pravatar.cc/64?img=2', meta: 'کارشناس IT', isOnline: false },
  { label: 'علی رضایی', value: 3, avatarUrl: 'https://i.pravatar.cc/64?img=3', meta: 'مهندس نرم‌افزار', isOnline: true },
  { label: 'زهرا کریمی', value: 4, avatarUrl: 'https://i.pravatar.cc/64?img=4', meta: 'مدیر پروژه', isOnline: true },
  { label: 'محمد حسینی', value: 5, avatarUrl: 'https://i.pravatar.cc/64?img=5', meta: 'کارشناس شبکه', isOnline: false },
  { label: 'نرگس صادقی', value: 6, avatarUrl: 'https://i.pravatar.cc/64?img=6', meta: 'طراح UI/UX', isOnline: true },
  { label: 'حسن مرادی', value: 7, avatarUrl: 'https://i.pravatar.cc/64?img=7', meta: 'مدیر سیستم', isOnline: false },
  { label: 'مریم نوری', value: 8, avatarUrl: 'https://i.pravatar.cc/64?img=8', meta: 'تحلیلگر داده', isOnline: true }
]);

const typeItems = [
  'درخواست', 
  'مشکل فنی', 
  'پیشنهاد', 
  'شکایت', 
  'سوال', 
  'درخواست آموزش',
  'درخواست تعمیر',
  'درخواست نصب'
];

const priorityItems = computed<ComboItem[]>(() => {
  const colors = ['#10b981','#84cc16','#f59e0b','#fb923c','#ef4444'];
  const labels = ['غیر اضطراری','کم','متوسط','بالا','اضطراری'];
  return labels.map((l,i)=>({ label:l, value:i+1, color:colors[i] }));
});

const form = reactive({
  building: null as null | number | string,
  service: null as null | number | string,
  receiver: null as null | string | number,
  priority: 3 as number,
  type: null as null | number | string,
  contact: '',
  title: '',
  descriptionHtml: '',
});

// Helper functions to get actual values instead of indices
function getBuildingName(id: any) {
  if (!id) return null;
  const item = buildingItems.value.find(item => item.value === id);
  return item ? item.label : id;
}

function getServiceName(id: any) {
  if (!id) return null;
  const item = serviceItems.value.find(item => item.value === id);
  return item ? item.label : id;
}

function getReceiverName(id: any) {
  if (!id) return null;
  const item = receiverItems.value.find(item => item.value === id);
  return item ? item.label : id;
}

function getPriorityName(id: any) {
  if (!id) return null;
  const item = priorityItems.value.find(item => item.value === id);
  return item ? item.label : id;
}

function getTypeName(id: any) {
  if (!id) return null;
  const item = typeItems.find((item: any) => item === id);
  return item || id;
}

// Computed properties for better data access
const formData = computed(() => ({
  title: form.title,
  building: {
    id: form.building,
    name: getBuildingName(form.building)
  },
  service: {
    id: form.service,
    name: getServiceName(form.service)
  },
  receiver: {
    id: form.receiver,
    name: getReceiverName(form.receiver)
  },
  priority: {
    id: form.priority,
    name: getPriorityName(form.priority)
  },
  type: {
    id: form.type,
    name: getTypeName(form.type)
  },
  contact: form.contact,
  description: form.descriptionHtml,
  checklist: checklist.value,
  files: files.value
}));

// Validation errors
const errors = reactive({
  title: '',
  building: '',
  service: '',
  receiver: '',
  type: '',
  priority: '',
  contact: '',
  description: ''
});

// Phone number validation - format: 09 followed by 9 digits
const phoneRegex = /^09\d{9}$/;

// Validation functions
function validateTitle() {
  if (!form.title || form.title.trim().length < 5) {
    errors.title = 'عنوان تیکت باید حداقل 5 کاراکتر باشد';
    return false;
  }
  errors.title = '';
  return true;
}

function validateBuilding() {
  if (!form.building) {
    errors.building = 'انتخاب بخش اجباری است';
    return false;
  }
  errors.building = '';
  return true;
}

function validateService() {
  if (!form.service) {
    errors.service = 'انتخاب سرویس اجباری است';
    return false;
  }
  errors.service = '';
  return true;
}

function validateReceiver() {
  if (!form.receiver) {
    errors.receiver = 'انتخاب گیرنده اجباری است';
    return false;
  }
  errors.receiver = '';
  return true;
}

function validateType() {
  if (!form.type) {
    errors.type = 'انتخاب نوع اجباری است';
    return false;
  }
  errors.type = '';
  return true;
}

function validatePriority() {
  if (!form.priority) {
    errors.priority = 'انتخاب وضعیت اجباری است';
    return false;
  }
  errors.priority = '';
  return true;
}

function formatPhoneNumber(value: string) {
  // Remove all non-digit characters
  const digits = value.replace(/\D/g, '');
  
  // If it doesn't start with 09, force it to start with 09
  let formatted = digits;
  if (digits.length > 0 && !digits.startsWith('09')) {
    // If user starts typing with other numbers, prepend 09
    if (digits.length === 1 && digits[0] !== '0') {
      formatted = '09' + digits;
    } else if (digits.length >= 2 && !digits.startsWith('09')) {
      // If user types something like 123..., replace with 09123...
      formatted = '09' + digits.slice(1);
    }
  }
  
  // Limit to 11 digits (09 + 9 more digits)
  const limitedDigits = formatted.slice(0, 11);
  
  return limitedDigits;
}

function onPhoneInput(event: Event) {
  const target = event.target as HTMLInputElement;
  const formatted = formatPhoneNumber(target.value);
  form.contact = formatted;
  validateContact(); // Validate immediately during input
}

function validateContact() {
  if (!form.contact) {
    errors.contact = 'شماره تماس اجباری است';
    return false;
  }
  
  // Live validation - check as user types
  if (form.contact.length > 0 && !form.contact.startsWith('09')) {
    errors.contact = 'شماره تماس باید با 09 شروع شود';
    return false;
  }
  
  if (form.contact.length > 2 && form.contact.length < 11) {
    errors.contact = `شماره تماس باید 11 رقم باشد (${form.contact.length}/11)`;
    return false;
  }
  
  if (form.contact.length === 11 && !phoneRegex.test(form.contact)) {
    errors.contact = 'شماره تماس باید به فرمت 09123456789 باشد';
    return false;
  }
  
  // Only clear error if it's complete and valid
  if (form.contact.length === 11 && phoneRegex.test(form.contact)) {
    errors.contact = '';
    return true;
  }
  
  // For incomplete but valid start, show progress
  if (form.contact.startsWith('09') && form.contact.length < 11) {
    errors.contact = `شماره تماس باید 11 رقم باشد (${form.contact.length}/11)`;
    return false;
  }
  
  return false;
}

function validateDescription() {
  const textContent = editorRef.value?.textContent || '';
  if (textContent.trim().length < 30) {
    errors.description = 'توضیحات باید حداقل 30 کاراکتر باشد';
    return false;
  }
  errors.description = '';
  return true;
}

function validateAll() {
  const isValid = validateTitle() && 
                  validateBuilding() && 
                  validateService() && 
                  validateReceiver() && 
                  validateType() && 
                  validatePriority() && 
                  validateContact() && 
                  validateDescription();
  return isValid;
}

// Combo states
const serviceComboOpen = ref(false);
const receiverComboOpen = ref(false);
const typeComboOpen = ref(false);

// Combo functions
function toggleServiceCombo() { serviceComboOpen.value = !serviceComboOpen.value; }
function toggleReceiverCombo() { receiverComboOpen.value = !receiverComboOpen.value; }
function toggleTypeCombo() { typeComboOpen.value = !typeComboOpen.value; }

function closeServiceCombo() { serviceComboOpen.value = false; }
function closeReceiverCombo() { receiverComboOpen.value = false; }
function closeTypeCombo() { typeComboOpen.value = false; }

function selectService(value: string | number) {
  form.service = value;
  validateService();
  closeServiceCombo();
}

function selectReceiver(value: string | number) {
  form.receiver = value;
  validateReceiver();
  closeReceiverCombo();
}

function selectType(value: string | number) {
  form.type = value;
  validateType();
  closeTypeCombo();
}

// Selected items for display
const selectedService = computed(() => serviceItems.value.find(item => item.value === form.service) || null);
const selectedReceiver = computed(() => receiverItems.value.find(item => item.value === form.receiver) || null);
const selectedType = computed(() => typeItems.find(item => item === form.type) || null);

// checklist logic
const checklist = ref<string[]>([]);
const newItem = ref<string>('');
function addItem(){
  const t = newItem.value.trim();
  if(!t) return; checklist.value.push(t); newItem.value = '';
}
function removeChecklistAt(i: number){
  checklist.value.splice(i, 1);
}
function clearChecklist(){
  checklist.value = [];
}
function onNewKey(e: KeyboardEvent){
  if(e.key === 'Enter'){ e.preventDefault(); addItem(); }
  if(e.key === 'Backspace' && newItem.value === '' && checklist.value.length){
    checklist.value.pop();
  }
}
// drag & drop
let dragIndex = -1;
const overIndex = ref<number | null>(null);
function onDragStart(i:number){ dragIndex = i; }
function onDragEnter(i:number){ overIndex.value = i; }
function onDragEnd(){ dragIndex = -1; overIndex.value = null; }
function onDrop(i:number){
  if(dragIndex === -1 || dragIndex === i) { overIndex.value = null; return; }
  const arr = checklist.value.slice();
  const [m] = arr.splice(dragIndex,1);
  arr.splice(i,0,m);
  checklist.value = arr; dragIndex = -1; overIndex.value = null;
}

// rich text (advanced toolbar around contenteditable)
const editorRef = ref<HTMLDivElement | null>(null);
function exec(cmd: string, value?: string){ document.execCommand(cmd, false, value); syncHtml(); }
function syncHtml(){ form.descriptionHtml = editorRef.value?.innerHTML ?? ''; }
function setHeading(level: 1|2|3){ document.execCommand('formatBlock', false, 'H' + level); syncHtml(); }
function setParagraph(){ document.execCommand('formatBlock', false, 'P'); syncHtml(); }
function insertQuote(){ document.execCommand('formatBlock', false, 'BLOCKQUOTE'); syncHtml(); }
function setCode(){ document.execCommand('formatBlock', false, 'PRE'); syncHtml(); }
const fore = ref<string>('#111827');
const back = ref<string>('#ffffff');
function setForeColor(color: string){ document.execCommand('foreColor', false, color); syncHtml(); }
function setBackColor(color: string){ document.execCommand('hiliteColor', false, color); syncHtml(); }
function undo(){ document.execCommand('undo'); }
function redo(){ document.execCommand('redo'); }
function removeFormatting(){ document.execCommand('removeFormat'); syncHtml(); }

// attachments via UploadModal
const showUploader = ref<boolean>(false);
const files = ref<File[]>([]);
const previews = ref<Record<string, string>>({});
function openUpload(){ showUploader.value = true; }
function openUploadForPreview(){ showUploader.value = true; }
function onFilesPassed(list: File[]){ files.value = list.slice(); showUploader.value = false; }
function isImage(f: File){ return f.type.startsWith('image/'); }
function formatBytes(bytes: number){
  if(bytes === 0) return '0 B';
  const k = 1024, sizes = ['B','KB','MB','GB','TB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return `${(bytes / Math.pow(k, i)).toFixed(1)} ${sizes[i]}`;
}
function ensurePreviews(){
  const next: Record<string,string> = { ...previews.value };
  for(const f of files.value){
    if(isImage(f) && !next[f.name]){
      next[f.name] = URL.createObjectURL(f);
    }
  }
  for(const k of Object.keys(next)){
    if(!files.value.find(f => f.name === k)){
      URL.revokeObjectURL(next[k]);
      delete next[k];
    }
  }
  previews.value = next;
}
watch(files, ensurePreviews, { deep: true });
function removeAttachment(name: string){
  files.value = files.value.filter(f => f.name !== name);
}
onBeforeUnmount(()=>{
  for(const k of Object.keys(previews.value)) URL.revokeObjectURL(previews.value[k]);
});

function submit(){
  if (!validateAll()) {
    return; // Don't submit if validation fails
  }
  
  // Emit the structured data
  emit('submit', formData.value);
  
  // Save to JSON file (optional)
  saveToJsonFile(formData.value);
  
  isOpen.value = false;
}

function saveToJsonFile(data: any) {
  try {
    // Create a data object with timestamp
    const jsonData = {
      timestamp: new Date().toISOString(),
      ticketData: data
    };
    
    // Convert to JSON string with proper formatting
    const jsonString = JSON.stringify(jsonData, null, 2);
    
    // Create a blob with the JSON data
    const blob = new Blob([jsonString], { type: 'application/json' });
    
    // Create a download link
    const url = URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.href = url;
    link.download = `ticket_${new Date().getTime()}.json`;
    
    // Trigger download
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    
    // Clean up the URL object
    URL.revokeObjectURL(url);
    
    console.log('Ticket data saved to JSON file:', jsonData);
  } catch (error) {
    console.error('Error saving JSON file:', error);
  }
}
function cancel(){ emit('cancel'); isOpen.value = false; }

// Checklist open/close on outside click
const isChecklistOpen = ref<boolean>(true);
const checklistRef = ref<HTMLDivElement | null>(null);
function toggleChecklist(){ isChecklistOpen.value = !isChecklistOpen.value; }
function onDocClick(e: MouseEvent){
  if(!checklistRef.value) return;
  if(!checklistRef.value.contains(e.target as Node)){
    isChecklistOpen.value = false;
  }
}
// Combo click outside handlers
function onComboClickOutside(e: MouseEvent) {
  const target = e.target as HTMLElement;
  if (!target.closest('.combo-service')) closeServiceCombo();
  if (!target.closest('.combo-receiver')) closeReceiverCombo();
  if (!target.closest('.combo-type')) closeTypeCombo();
}

onMounted(()=>{ 
  document.addEventListener('click', onDocClick);
  document.addEventListener('click', onComboClickOutside);
  // Initialize dialog if it should be open
  if (isOpen.value && dialogRef.value) {
    dialogRef.value.showModal();
  }
});
onBeforeUnmount(()=>{ 
  document.removeEventListener('click', onDocClick);
  document.removeEventListener('click', onComboClickOutside);
});

// Theme object for UploadModal (blue sky cold gradient)
const uploadTheme = {
  primary: '#4E71FF',
  onPrimary: '#ffffff',
  surface: '#f0f9ff',
  text: '#2E073F',
  border: 'transparent',
  overlay: 'rgba(78, 113, 255, 0.1)',
  canvas: '#ffffff',
  danger: '#ef4444'
};

</script>

<template>
  <dialog 
    ref="dialogRef" 
    class="ticket-dialog" 
    @close="handleDialogClose"
    @click.self="cancel"
  >
    <div class="modal">
        <div class="header">
          <div class="title">ایجاد تیکت</div>
          <button class="x" @click="cancel">×</button>
        </div>

        <div class="content">
          <div class="form">

          <div class="field full">
            <label>عنوان تیکت <span class="required">*</span></label>
            <input class="input" :class="{ error: errors.title }" v-model="form.title" @input="validateTitle" @blur="validateTitle" placeholder="عنوان تیکت را وارد کنید" />
            <div v-if="errors.title" class="error-message">{{ errors.title }}</div>
          </div>

          <div class="field full">
            <label>بخش <span class="required">*</span></label>
            <select class="select" :class="{ error: errors.building }" v-model="form.building" @change="validateBuilding">
              <option disabled :value="null">بخش مورد نظر</option>
              <option v-for="b in buildingItems" :key="b.value" :value="b.value">{{ b.label }}</option>
            </select>
            <div v-if="errors.building" class="error-message">{{ errors.building }}</div>
          </div>

          <div class="row-2">
            <div class="field">
              <label>وضعیت <span class="required">*</span></label>
              <div class="chips">
                <button v-for="p in priorityItems" :key="p.value" class="chip" :class="{ active: form.priority===p.value }" @click="form.priority = Number(p.value); validatePriority()">
                  <span class="dot" :style="{ background:p.color }"></span>
                  {{ p.label }}
                </button>
              </div>
              <div v-if="errors.priority" class="error-message">{{ errors.priority }}</div>
            </div>
            <div class="field">
              <label>شماره تماس <span class="required">*</span></label>
              <input class="input" :class="{ error: errors.contact }" v-model="form.contact" @input="onPhoneInput" @blur="validateContact" placeholder="09123456789" />
              <div v-if="errors.contact" class="error-message">{{ errors.contact }}</div>
            </div>
          </div>

          <div class="row-3 combos-row">
            <div class="field combo-service">
              <label>سرویس <span class="required">*</span></label>
              <div class="combo-control" :class="{ error: errors.service }" @click.stop="toggleServiceCombo">
                <template v-if="selectedService">
                  <span class="text">{{ selectedService.label }}</span>
                </template>
                <span v-else class="placeholder">انتخاب کنید</span>
                <span class="chev">▾</span>
              </div>
              <div v-if="serviceComboOpen" class="combo-menu">
                <div v-for="item in serviceItems" :key="String(item.value)" class="combo-item" @click.stop="selectService(item.value)">
                  <span class="text">{{ item.label }}</span>
                </div>
              </div>
              <div v-if="errors.service" class="error-message">{{ errors.service }}</div>
            </div>
            <div class="field combo-receiver">
              <label>گیرنده <span class="required">*</span></label>
              <div class="combo-control" :class="{ error: errors.receiver }" @click.stop="toggleReceiverCombo">
                <template v-if="selectedReceiver">
                  <div class="avatar-container">
                    <img v-if="selectedReceiver.avatarUrl" class="avatar" :src="selectedReceiver.avatarUrl" alt="" />
                    <div v-if="selectedReceiver.isOnline" class="online-indicator"></div>
                  </div>
                  <span class="text">{{ selectedReceiver.label }}</span>
                </template>
                <span v-else class="placeholder">انتخاب کنید</span>
                <span class="chev">▾</span>
              </div>
              <div v-if="receiverComboOpen" class="combo-menu">
                <div v-for="item in receiverItems" :key="String(item.value)" class="combo-item" @click.stop="selectReceiver(item.value)">
                  <div class="avatar-container">
                    <img v-if="item.avatarUrl" class="avatar" :src="item.avatarUrl" alt="" />
                    <div v-if="item.isOnline" class="online-indicator"></div>
                  </div>
                  <div class="item-info">
                    <span class="text">{{ item.label }}</span>
                    <span v-if="item.meta" class="meta">{{ item.meta }}</span>
                  </div>
                </div>
              </div>
              <div v-if="errors.receiver" class="error-message">{{ errors.receiver }}</div>
            </div>
            <div class="field combo-type">
              <label>نوع <span class="required">*</span></label>
              <div class="combo-control" :class="{ error: errors.type }" @click.stop="toggleTypeCombo">
                <template v-if="selectedType">
                  <span class="text">{{ selectedType }}</span>
                </template>
                <span v-else class="placeholder">انتخاب کنید</span>
                <span class="chev">▾</span>
              </div>
              <div v-if="typeComboOpen" class="combo-menu">
                <div v-for="item in typeItems" :key="String(item)" class="combo-item" @click.stop="selectType(item)">
                  <span class="text">{{ item }}</span>
                </div>
              </div>
              <div v-if="errors.type" class="error-message">{{ errors.type }}</div>
            </div>
          </div>

          <div class="field full" ref="checklistRef">
            <button type="button" class="cl-toggle" @click="toggleChecklist">
              <span>چک‌لیست</span>
              <span class="chev" :class="{ open: isChecklistOpen }">▾</span>
            </button>
            <div v-if="isChecklistOpen" class="checklist v">
              <div class="cl-list">
                <div v-for="(c,i) in checklist" :key="c + i" class="cl-item" draggable="true"
                  @dragstart="onDragStart(i)" @dragenter.prevent="onDragEnter(i)" @dragover.prevent @dragend="onDragEnd" @drop="onDrop(i)" :class="{ over: overIndex===i }">
                  <span class="dot"></span>
                  <span class="text">{{ c }}</span>
                  <button class="cl-del" title="حذف" @click.stop="removeChecklistAt(i)">حذف</button>
                </div>
              </div>
              <div class="cl-actions">
                <input class="input cl-input" v-model="newItem" placeholder="آیتم جدید... (Enter)" @keydown="onNewKey" />
                <button class="btn cl-add-btn" @click="addItem">افزودن</button>
                <button class="btn danger cl-clear-btn" @click="clearChecklist" v-if="checklist.length">حذف همه</button>
              </div>
            </div>
          </div>

          <div class="field full">
            <label>فایل‌های ضمیمه</label>
            <div class="attachments-row">
              <button class="btn light" @click="openUpload">افزودن فایل</button>
              <div class="thumb-row" v-if="files.length">
                <div v-for="f in files" :key="f.name" class="thumb clickable" :title="`${f.name} \n${formatBytes(f.size)}`" @click="openUploadForPreview()">
                  <img v-if="isImage(f)" :src="previews[f.name]" alt="" />
                  <span v-else class="thumb-ext">{{ (f.name.split('.').pop() || '').toUpperCase() }}</span>
                  <button class="thumb-del" @click.stop="removeAttachment(f.name)">×</button>
                </div>
              </div>
            </div>
          </div>

          <div class="field full">
            <label>توضیحات <span class="required">*</span></label>
            <div class="toolbar">
              <button @click="exec('bold')" title="Bold"><b>B</b></button>
              <button @click="exec('italic')" title="Italic"><i>I</i></button>
              <button @click="exec('underline')" title="Underline"><u>U</u></button>
              <button @click="exec('strikeThrough')" title="Strike">S</button>
              <button @click="setHeading(1)" title="H1">H1</button>
              <button @click="setHeading(2)" title="H2">H2</button>
              <button @click="setHeading(3)" title="H3">H3</button>
              <button @click="setParagraph()" title="Paragraph">P</button>
              <button @click="exec('justifyRight')" title="Align Right">⟸</button>
              <button @click="exec('justifyCenter')" title="Align Center">⇔</button>
              <button @click="exec('justifyLeft')" title="Align Left">⟹</button>
              <button @click="exec('justifyFull')" title="Justify">≡</button>
              <button @click="exec('insertUnorderedList')" title="Bulleted List">•</button>
              <button @click="exec('insertOrderedList')" title="Numbered List">1.</button>
              <button @click="exec('outdent')" title="Outdent">⇤</button>
              <button @click="exec('indent')" title="Indent">⇥</button>
              <button @click="undo" title="Undo">↶</button>
              <button @click="redo" title="Redo">↷</button>
              <button @click="removeFormatting" title="Clear">✕</button>
              <button @click="insertQuote" title="Quote">❝❞</button>
              <button @click="setCode" title="Code">{ }</button>
            </div>
            <div class="editor" :class="{ error: errors.description }" ref="editorRef" contenteditable @input="syncHtml(); validateDescription()" placeholder="توضیحات را وارد کنید (حداقل 30 کاراکتر)"></div>
            <div v-if="errors.description" class="error-message">{{ errors.description }}</div>
          </div>

          </div>
        </div>

        <div class="footer">
          <button class="btn" @click="cancel">انصراف</button>
          <button class="btn primary" @click="submit">ایجاد تیکت</button>
        </div>
      </div>

      <UploadModal v-model:isOpen="showUploader" :selectedFiles="files" @files-passed="onFilesPassed" :theme="uploadTheme" />
  </dialog>
</template>

<style scoped>
.ticket-dialog{ 
  position: fixed; 
  inset: 0; 
  margin: auto; 
  padding: 0; 
  border: none; 
  background: transparent; 
  z-index: 1000;
  width: 90%;
  height: 90%;
  max-width: 1000px;
  max-height: 800px;
}

.ticket-dialog::backdrop {
  background: rgba(0, 0, 0, 0.18);
  backdrop-filter: blur(12px) saturate(1.2);
  -webkit-backdrop-filter: blur(12px) saturate(1.2);
}

/* Fallback for browsers that don't support backdrop-filter */
@supports not (backdrop-filter: blur(12px)) {
  .ticket-dialog::backdrop {
    background: rgba(0, 0, 0, 0.3);
  }
}

.modal{ 
  --bg:#ffffff; 
  --accent:#4E71FF; 
  --surface:#f0f9ff; 
  --border:transparent; 
  --text:#2E073F; 
  --muted:#2E073F; 
  width: 100%; 
  height: 100%; 
  background:var(--bg); 
  border-radius:16px; 
  box-shadow:0 16px 40px rgba(78, 113, 255, 0.15); 
  overflow:hidden; 
  color:var(--text); 
  display:flex; 
  flex-direction:column; 
  font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial; 
  font-weight: 500; 
  -webkit-font-smoothing: antialiased; 
  -moz-osx-font-smoothing: grayscale;
}
.header{ display:flex; align-items:center; padding:14px 16px }
.title{ font-weight: 600; letter-spacing:-0.01em; font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial }
.x{ margin-inline-start:auto; background:transparent; border:none; font-size:22px; line-height:22px; cursor:pointer; color:var(--muted) }
.card{ margin:16px; padding:12px; background:var(--surface); border-radius:12px }
.grow{ flex:1 }
.content{ padding:0 16px 16px 16px; overflow:auto }
.form{ display:grid; grid-template-columns:1fr; gap:12px; align-items:start }
.row-2{ display:grid; grid-template-columns:minmax(300px,1fr) 450px; gap:4px }
.row-2 > .field{ min-width:0 }
.row-3{ display:grid; grid-template-columns:1fr 1fr 1fr; gap:12px }
.combos-row .select.combo{ height:34px; border-radius:10px }
.field label{ display:block; font-size:12px; color:var(--muted); margin-bottom:6px; font-weight: 300; letter-spacing:.1px; font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial }
.field.full{ grid-column: 1 / -1 }
.input, .select{ width:100%; height:36px; background:#f8fafc; border:none; border-radius:12px; padding:0 12px; outline:none; box-sizing:border-box; font-size:14px; color:#2E073F; font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial; font-weight: 400 }
.input:focus, .select:focus{ box-shadow:0 0 0 3px rgba(78, 113, 255, 0.15) }
.input.error, .select.error{ border:2px solid #ef4444; background:#fef2f2 }
.editor.error{ border:2px solid #ef4444; background:#fef2f2 }
.required{ color:#ef4444; font-weight:600 }
.error-message{ color:#ef4444; font-size:12px; margin-top:4px; font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial }
.chips{ display:flex; flex-wrap:wrap; gap:8px }
.chip{ height:30px; padding:0 12px; border:none; border-radius:10px; background:#f8fafc; cursor:pointer; display:inline-flex; align-items:center; gap:8px; color:var(--muted); font-weight: 300; font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial }
.chip .dot{ width:10px; height:10px; border-radius:50% }
.chip.active{ color:#2E073F; background:#dbeafe; box-shadow:0 0 0 3px rgba(78, 113, 255, 0.15); font-weight: 500 }
.checklist.v{ display:flex; flex-direction:column; gap:8px }
.cl-toggle{ display:inline-flex; align-items:center; gap:8px; height:34px; padding:0 12px; background:var(--surface); border:none; border-radius:10px; cursor:pointer; color:var(--text); font-weight: 500; font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial }
.cl-toggle .chev{ transition:transform .15s ease }
.cl-toggle .chev.open{ transform:rotate(180deg) }
.cl-list{ display:flex; flex-direction:column; gap:6px; max-height:220px; overflow:auto; padding:6px; border-radius:12px; background:#fff }
.cl-item{ display:flex; align-items:center; gap:8px; padding:8px 10px; border:none; border-radius:10px; background:var(--surface); font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial; font-weight: 400 }
.cl-item.over{ background:#dbeafe }
.cl-item .dot{ width:8px; height:8px; border-radius:50%; background:#94a3b8 }
.cl-item .cl-del{ margin-inline-start:auto; background:#ff6b6b; color:#ffffff; border:none; border-radius:8px; padding:4px 8px; cursor:pointer; font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial; font-weight: 300; white-space: nowrap; font-size: 12px }
.cl-actions{ display:flex; align-items:center; gap:8px }
.cl-input{ flex: 0 0 200px; min-width: 650px; max-width: 90%;}
.cl-add-btn{ flex: 0 0 auto;max-width: 100px; min-width: 100px }
.cl-clear-btn{ flex: 1; min-width: 120px }
.toolbar{ display:flex; gap:6px; margin-bottom:6px; flex-wrap:wrap; background:var(--surface); border-radius:10px; padding:8px }
.toolbar button{ height:28px; padding:0 10px; border:none; background:#fff; border-radius:8px; cursor:pointer; font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial; color:#2E073F; font-weight: 400; transition:all 0.2s ease }
.toolbar button:hover{ background:var(--accent); color:#ffffff }
.editor{ min-height:280px; border:none; border-radius:12px; padding:12px; line-height:1.9; background:#f8fafc; color:#2E073F; font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial; font-weight: 400 }
.attachments-row{ display:flex; align-items:center; gap:10px; flex-wrap:wrap }
.thumb-row{ display:flex; align-items:center; gap:8px; flex-wrap:wrap }
.thumb{ position:relative; width:52px; height:52px; background:#fff; border:none; border-radius:10px; display:flex; align-items:center; justify-content:center; overflow:hidden; cursor:pointer; transition:all 0.2s ease }
.thumb:hover{ transform:scale(1.05); box-shadow:0 4px 12px rgba(78, 113, 255, 0.2) }
.thumb img{ width:100%; height:100%; object-fit:cover }
.thumb-ext{ font-size:11px; color:#64748b }
.thumb-del{ position:absolute; top:-6px; right:-6px; width:18px; height:18px; border:none; border-radius:30%; background:#0f172a; color:#fff; cursor:pointer; line-height:18px; padding:7px 12px 18px 15px }
.thumbs{ margin-top:8px; display:flex; flex-direction:column; gap:6px }
.file{ display:flex; align-items:center; gap:8px; border:none; border-radius:10px; padding:6px 10px; background:#fff }
.file .name{ flex:1; overflow:hidden; text-overflow:ellipsis; white-space:nowrap }
.file .del{ background:#fee2e2; color:#991b1b; border:none; border-radius:8px; padding:4px 8px; cursor:pointer }
.footer{ padding:12px 16px; background:var(--surface); display:flex; justify-content:flex-end; gap:8px }
.btn{ height:34px; padding:0 12px; border-radius:10px; border:none; background:#f8fafc; cursor:pointer; font-weight: 500; color:var(--text); font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial }
.btn.primary{ background:#4E71FF; color:#ffffff; font-weight: 600 }
.btn.danger{ background:#ff6b6b; color:#ffffff; font-weight: 500 }

/* Combo Styles */
.field.combo-service,
.field.combo-receiver,
.field.combo-type {
  position: relative;
}

.combo-control{ 
  display:flex; 
  align-items:center; 
  gap:8px; 
  min-height:36px; 
  border:1px solid #e5e7eb; 
  border-radius:10px; 
  padding:4px 12px; 
  background:#f8fafc; 
  cursor:pointer; 
  user-select:none; 
  font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial;
  transition: all 0.2s ease;
}
.combo-control:hover{ background:#f1f5f9; border-color:#cbd5e1; }
.combo-control.error{ border:2px solid #ef4444; background:#fef2f2; }
.combo-control .avatar-container{ position: relative; display: inline-block; }
.combo-control .avatar{ width:22px; height:22px; border-radius:50% }
.combo-control .online-indicator{ 
  position: absolute; 
  bottom: 0; 
  right: 0; 
  width: 8px; 
  height: 8px; 
  background: #10b981; 
  border: 2px solid #fff; 
  border-radius: 50%; 
  box-shadow: 0 0 0 1px rgba(16, 185, 129, 0.3);
}
.combo-control .text{ flex:1; color:#2E073F; font-weight: 400 }
.combo-control .placeholder{ color:#9ca3af; flex:1 }
.combo-control .chev{ margin-inline-start:auto; color:#9ca3af; transition: transform 0.2s ease }
.combo-control:hover .chev{ color:#64748b }

.combo-menu{ 
  position:absolute; 
  inset-inline:0; 
  top:100%; 
  margin-top:6px; 
  background:#fff; 
  border:1px solid #e5e7eb; 
  border-radius:10px; 
  box-shadow:0 10px 30px rgba(0,0,0,.08); 
  padding:6px; 
  max-height:220px; 
  overflow:auto; 
  z-index:1000;
  font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial;
}
.combo-item{ 
  display:flex; 
  align-items:center; 
  gap:8px; 
  padding:8px; 
  border-radius:8px; 
  cursor:pointer; 
  transition: background-color 0.2s ease;
  font-weight: 400;
}
.combo-item:hover{ background:#f8fafc }
.combo-item .avatar-container{ position: relative; display: inline-block; }
.combo-item .avatar{ width:24px; height:24px; border-radius:50% }
.combo-item .online-indicator{ 
  position: absolute; 
  bottom: 0; 
  right: 0; 
  width: 8px; 
  height: 8px; 
  background: #10b981; 
  border: 2px solid #fff; 
  border-radius: 50%; 
  box-shadow: 0 0 0 1px rgba(16, 185, 129, 0.3);
}
.combo-item .item-info{ 
  display: flex; 
  flex-direction: column; 
  gap: 2px; 
  flex: 1; 
}
.combo-item .text{ color:#2E073F; font-weight: 500; }
.combo-item .meta{ color:#6b7280; font-size: 12px; font-weight: 400; }

@media (max-width: 980px){ .modal{ width:96vw } }
@media (max-width: 820px){ .row-2{ grid-template-columns:1fr } .row-3{ grid-template-columns:1fr } }
</style>


