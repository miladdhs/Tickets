<script setup lang="ts">
import { ref, computed, watch, onMounted, onBeforeUnmount } from 'vue';
import { PlusCircleOutlined, ZoomInOutlined, ZoomOutOutlined, ReloadOutlined, RotateLeftOutlined, RotateRightOutlined, CloseOutlined, LeftOutlined, RightOutlined, DeleteOutlined } from '@ant-design/icons-vue';

type MediaKind = 'image' | 'video' | 'audio' | 'document' | 'other';

const props = defineProps<{
  isOpen: boolean;
  selectedFiles?: File[];
  allowedExtensions?: string | string[];
  allowedTypes?: 'image' | 'video' | Array<'image' | 'video'>;
  maxFileSizeMB?: number;
  theme?: {
    primary?: string;
    onPrimary?: string;
    surface?: string;
    text?: string;
    border?: string;
    overlay?: string;
    canvas?: string;
    danger?: string;
  };
}>();

const emit = defineEmits<{
  (e: 'update:isOpen', value: boolean): void;
  (e: 'update:selectedFiles', files: File[]): void;
  (e: 'file-error', message: string): void;
  (e: 'files-passed', files: File[]): void;
}>();

const isOpen = ref<boolean>(props.isOpen);
const dialogRef = ref<HTMLDialogElement | null>(null);

watch(() => props.isOpen, v => {
  isOpen.value = !!v;
  if (dialogRef.value) {
    if (v) {
      dialogRef.value.showModal();
    } else {
      dialogRef.value.close();
    }
  }
});

watch(isOpen, v => emit('update:isOpen', v));

const files = ref<File[]>(props.selectedFiles ? [...props.selectedFiles] : []);
watch(() => props.selectedFiles, v => (files.value = v ? [...v] : []));
function updateFiles(next: File[]){ files.value = next; emit('update:selectedFiles', [...next]); }

const currentIndex = ref<number>(0);
const currentFile = computed(() => files.value[currentIndex.value] ?? null);
const objectUrl = ref<string | null>(null);
const mediaKind = ref<MediaKind | null>(null);
const viewerRef = ref<HTMLElement | null>(null);
const mediaRef = ref<HTMLImageElement | HTMLVideoElement | HTMLAudioElement | null>(null);
const textContent = ref<string>('');
const isLoadingText = ref<boolean>(false);

const maxBytes = computed(() => Math.max(1, Math.floor((props.maxFileSizeMB ?? 10) * 1024 * 1024)));
const extensions = computed(() => {
  if (!props.allowedExtensions) return ['png','jpg','jpeg','webp','gif','bmp','svg','mp4','webm','ogg'];
  return Array.isArray(props.allowedExtensions) ? props.allowedExtensions : props.allowedExtensions.split(',').map(s => s.trim()).filter(Boolean);
});
const acceptList = computed(() => {
  // 1) If explicit extensions provided, honor them (highest precedence)
  if (props.allowedExtensions && (Array.isArray(props.allowedExtensions) ? props.allowedExtensions.length : String(props.allowedExtensions).length)) {
    const list = Array.isArray(props.allowedExtensions)
      ? props.allowedExtensions
      : String(props.allowedExtensions).split(',').map(s => s.trim()).filter(Boolean);
    return list.map(ext => ext.startsWith('.') ? ext : `.${ext}`);
  }
  // 2) Otherwise use broad types (image/* or video/*)
  if (props.allowedTypes && ((Array.isArray(props.allowedTypes) && props.allowedTypes.length) || !Array.isArray(props.allowedTypes))) {
    const list = Array.isArray(props.allowedTypes) ? props.allowedTypes : [props.allowedTypes];
    return list.map(t => t === 'image' ? 'image/*' : 'video/*');
  }
  // 3) Fallback: infer from default or allowedExtensions computed
  const map: Record<string, string> = {
    png: 'image/png', jpg: 'image/jpeg', jpeg: 'image/jpeg', webp: 'image/webp', gif: 'image/gif', bmp: 'image/bmp', svg: 'image/svg+xml',
    mp4: 'video/mp4', webm: 'video/webm', ogg: 'video/ogg'
  };
  return extensions.value.map(ext => map[ext.toLowerCase()] ?? `.${ext}`);
});

// Theming: map provided theme to CSS variables so the whole modal inherits unified colors
const rootVars = computed(() => {
  const t = props.theme ?? {};
  return {
    '--mbox-accent': t.primary ?? '#4E71FF',
    '--mbox-on-accent': t.onPrimary ?? '#ffffff',
    '--mbox-panel-bg': t.surface ?? '#f0f9ff',
    '--mbox-text': t.text ?? '#2E073F',
    '--mbox-border': t.border ?? 'transparent',
    '--mbox-overlay-bg': t.overlay ?? 'rgba(78, 113, 255, 0.1)',
    '--mbox-canvas-bg': t.canvas ?? '#ffffff',
    '--mbox-danger': t.danger ?? '#ef4444'
  } as Record<string, string>;
});

function inferKind(f: File): MediaKind | null {
  if (f.type.startsWith('image/')) return 'image';
  if (f.type.startsWith('video/')) return 'video';
  if (f.type.startsWith('audio/')) return 'audio';
  if (f.type.startsWith('application/pdf') || f.type.startsWith('text/')) return 'document';
  const ext = f.name.split('.').pop()?.toLowerCase();
  if (!ext) return 'other';
  if (['jpg','jpeg','png','webp','gif','bmp','svg'].includes(ext)) return 'image';
  if (['mp4','webm','ogg','avi','mov','mkv'].includes(ext)) return 'video';
  if (['mp3','wav','ogg','aac','flac','m4a'].includes(ext)) return 'audio';
  if (['pdf','txt','doc','docx','rtf','md'].includes(ext)) return 'document';
  return 'other';
}

function validateFile(f: File): string | null {
  const ext = f.name.split('.').pop()?.toLowerCase() ?? '';
  const allowed = extensions.value.map(e => e.toLowerCase());
  const sizeOk = f.size <= maxBytes.value;
  const extOk = allowed.includes(ext);
  if (!extOk) return `فرمت فایل مجاز نیست: .${ext}`;
  if (!sizeOk) return `حجم فایل بیشتر از حد مجاز است (${Math.round(f.size/1024)}KB)`;
  return null;
}

function chooseFiles(){
  const input = document.createElement('input');
  input.type = 'file';
  input.accept = acceptList.value.join(',');
  input.multiple = true;
  input.onchange = () => {
    const selected = input.files ? Array.from(input.files) : [];
    if (!selected.length) return;
    const valid: File[] = [];
    for (const f of selected){
      const msg = validateFile(f);
      if (msg){ emit('file-error', msg); continue; }
      // prevent duplicates by file name
      const exists = files.value.some(x => x.name === f.name);
      if (exists) { continue; }
      valid.push(f);
    }
    if (!valid.length) return;
    updateFiles([...(files.value ?? []), ...valid]);
    if (files.value.length) currentIndex.value = files.value.length - 1;
  };
  input.click();
}

watch(currentFile, async (f) => {
  if (objectUrl.value) URL.revokeObjectURL(objectUrl.value);
  objectUrl.value = f ? URL.createObjectURL(f) : null;
  mediaKind.value = f ? inferKind(f) : null;
  textContent.value = '';
  isLoadingText.value = false;
  
  // Load text content for text files
  if (f && mediaKind.value === 'document' && f.type.startsWith('text/')) {
    isLoadingText.value = true;
    try {
      textContent.value = await readTextFile(f);
    } catch (error) {
      console.error('Failed to read text file:', error);
      textContent.value = 'خطا در خواندن فایل متنی';
    } finally {
      isLoadingText.value = false;
    }
  }
  
  resetTransform();
  setTimeout(() => fitToViewer(), 0);
});

// Zoom / Pan state
const scale = ref<number>(1);
const rotationDeg = ref<number>(0);
const offsetX = ref<number>(0);
const offsetY = ref<number>(0);
const minScale = 0.2;
const maxScale = 5;
const isDragging = ref<boolean>(false);
let dragStartX = 0, dragStartY = 0, originX = 0, originY = 0;

function resetTransform(){
  scale.value = 1; rotationDeg.value = 0; offsetX.value = 0; offsetY.value = 0;
}
function fitToViewer(){
  const viewer = viewerRef.value;
  const media = mediaRef.value as HTMLImageElement | HTMLVideoElement | null;
  if (!viewer || !media) return;
  const vw = viewer.clientWidth;
  const vh = viewer.clientHeight;
  let mw = 0, mh = 0;
  if (mediaKind.value === 'image'){
    const img = media as HTMLImageElement;
    if (!img.naturalWidth || !img.naturalHeight) return;
    mw = img.naturalWidth; mh = img.naturalHeight;
  } else if (mediaKind.value === 'video') {
    const vid = media as HTMLVideoElement;
    if (!vid.videoWidth || !vid.videoHeight) return;
    mw = vid.videoWidth; mh = vid.videoHeight;
  }
  if (mw && mh){
    const s = Math.min(vw / mw, vh / mh);
    scale.value = Math.max(minScale, Math.min(maxScale, s));
    offsetX.value = 0;
    offsetY.value = 0;
  }
}
function zoomIn(){ scale.value = Math.min(maxScale, scale.value + 0.1); offsetX.value = 0; offsetY.value = 0; }
function zoomOut(){ scale.value = Math.max(minScale, scale.value - 0.1); offsetX.value = 0; offsetY.value = 0; }
function rotateLeft(){ rotationDeg.value = (rotationDeg.value - 90 + 360) % 360; }
function rotateRight(){ rotationDeg.value = (rotationDeg.value + 90) % 360; }

function onWheel(e: WheelEvent){
  if (!currentFile.value) return;
  const prevScale = scale.value;
  const delta = e.deltaY < 0 ? 0.1 : -0.1;
  const next = Math.min(maxScale, Math.max(minScale, prevScale + delta));
  if (next === prevScale) return;
  scale.value = next;
  // auto-center content on zoom
  offsetX.value = 0;
  offsetY.value = 0;
}

function onMouseDown(e: MouseEvent){
  if (!currentFile.value) return;
  isDragging.value = true;
  dragStartX = e.clientX; dragStartY = e.clientY;
  originX = offsetX.value; originY = offsetY.value;
}
function onMouseMove(e: MouseEvent){
  if (!isDragging.value) return;
  const dx = e.clientX - dragStartX;
  const dy = e.clientY - dragStartY;
  offsetX.value = originX + dx;
  offsetY.value = originY + dy;
}
function onMouseUp(){ isDragging.value = false; }

onMounted(() => {
  window.addEventListener('mouseup', onMouseUp);
  // Initialize dialog if it should be open
  if (isOpen.value && dialogRef.value) {
    dialogRef.value.showModal();
  }
});
onBeforeUnmount(() => {
  window.removeEventListener('mouseup', onMouseUp);
  if (objectUrl.value) URL.revokeObjectURL(objectUrl.value);
});

function close(){ 
  isOpen.value = false; 
  if (dialogRef.value) {
    dialogRef.value.close();
  }
}

function handleDialogClose() {
  isOpen.value = false;
}
function removeCurrent(){
  if (!currentFile.value) return;
  const next = files.value.slice();
  next.splice(currentIndex.value, 1);
  updateFiles(next);
  if (currentIndex.value >= next.length) currentIndex.value = Math.max(0, next.length - 1);
}
function prev(){ if (currentIndex.value > 0) currentIndex.value--; }
function next(){ if (currentIndex.value < files.value.length - 1) currentIndex.value++; }

function clearAll(){
  updateFiles([]);
  currentIndex.value = 0;
  if (objectUrl.value) { try { URL.revokeObjectURL(objectUrl.value); } catch {} }
  objectUrl.value = null;
  mediaKind.value = null;
  resetTransform();
}

function confirmAndClose(){
  if (files.value.length > 0){
    emit('files-passed', [...files.value]);
  } else {
    emit('file-error', 'هیچ فایلی انتخاب نشده است');
  }
  clearAll();
  close();
}

const transformStyle = computed(() => `transform: translate(-50%, -50%) translate(${offsetX.value}px, ${offsetY.value}px) scale(${scale.value}) rotate(${rotationDeg.value}deg);`);

function createThumbUrl(file: File): string {
  const urlCtor = (window as any).URL || (window as any).webkitURL;
  return urlCtor.createObjectURL(file);
}
function onThumbLoaded(e: Event){
  const img = e.target as HTMLImageElement | null;
  if (!img) return;
  const urlCtor = (window as any).URL || (window as any).webkitURL;
  try { urlCtor.revokeObjectURL(img.src); } catch {}
}

function formatFileSize(bytes: number): string {
  if (bytes === 0) return '0 B';
  const k = 1024;
  const sizes = ['B', 'KB', 'MB', 'GB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(1)) + ' ' + sizes[i];
}

function getFileIcon(file: File): string {
  const ext = file.name.split('.').pop()?.toLowerCase() || '';
  const kind = inferKind(file);
  
  switch (kind) {
    case 'image': return '🖼️';
    case 'video': return '🎥';
    case 'audio': return '🎵';
    case 'document': 
      if (ext === 'pdf') return '📄';
      if (['txt', 'md', 'rtf'].includes(ext)) return '📝';
      if (['doc', 'docx'].includes(ext)) return '📄';
      return '📄';
    default: return '📁';
  }
}

function readTextFile(file: File): Promise<string> {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = (e) => resolve(e.target?.result as string || '');
    reader.onerror = () => reject(new Error('Failed to read file'));
    reader.readAsText(file);
  });
}
</script>

<template>
  <dialog 
    ref="dialogRef" 
    class="mbox-dialog" 
    :style="rootVars" 
    @close="handleDialogClose"
    @click.self="close"
  >
    <div class="mbox-modal">
        <div class="mbox-header">
          <div class="title">مدیریت آپلود</div>
          <div class="actions">
            <button class="icon-btn" @click="chooseFiles" title="افزودن"><PlusCircleOutlined /><span class="label">اضافه</span></button>
            <button class="icon-btn" :disabled="!currentFile" @click="zoomIn" title="بزرگ‌نمایی"><ZoomInOutlined /></button>
            <button class="icon-btn" :disabled="!currentFile" @click="zoomOut" title="کوچک‌نمایی"><ZoomOutOutlined /></button>
            <button class="icon-btn" :disabled="!currentFile" @click="rotateLeft" title="چرخش چپ"><RotateLeftOutlined /></button>
            <button class="icon-btn" :disabled="!currentFile" @click="rotateRight" title="چرخش راست"><RotateRightOutlined /></button>
            <button class="icon-btn" :disabled="!currentFile" @click="resetTransform" title="بازنشانی"><ReloadOutlined /></button>
            <button class="icon-btn danger" :disabled="!currentFile" @click="removeCurrent" title="حذف"><DeleteOutlined /></button>
            <button class="icon-btn" @click="close" title="بستن"><CloseOutlined /></button>
          </div>
        </div>

        <div class="mbox-body">
          <div class="viewer" ref="viewerRef" @wheel.passive="onWheel" @mousedown="onMouseDown" @mousemove="onMouseMove">
            <!-- Image Preview -->
            <template v-if="currentFile && objectUrl && mediaKind === 'image'">
              <img ref="mediaRef" :src="objectUrl" :style="transformStyle" alt="preview" draggable="false" class="centered" @load="fitToViewer" />
            </template>
            
            <!-- Video Preview -->
            <template v-else-if="currentFile && objectUrl && mediaKind === 'video'">
              <video ref="mediaRef" :src="objectUrl" controls :style="transformStyle" draggable="false" class="centered" @loadedmetadata="fitToViewer"></video>
            </template>
            
            <!-- Audio Preview -->
            <template v-else-if="currentFile && objectUrl && mediaKind === 'audio'">
              <div class="audio-preview">
                <div class="audio-icon">🎵</div>
                <audio ref="mediaRef" :src="objectUrl" controls class="audio-player"></audio>
                <div class="file-info">
                  <div class="file-name">{{ currentFile.name }}</div>
                  <div class="file-size">{{ formatFileSize(currentFile.size) }}</div>
                </div>
              </div>
            </template>
            
            <!-- Document Preview -->
            <template v-else-if="currentFile && mediaKind === 'document'">
              <div class="document-preview">
                <div class="document-icon">{{ getFileIcon(currentFile) }}</div>
                <div class="file-info">
                  <div class="file-name">{{ currentFile.name }}</div>
                  <div class="file-size">{{ formatFileSize(currentFile.size) }}</div>
                  <div class="file-type">{{ currentFile.type || 'نوع فایل نامشخص' }}</div>
                </div>
                
                <!-- Text content preview for text files -->
                <div v-if="currentFile.type.startsWith('text/')" class="text-content">
                  <div v-if="isLoadingText" class="loading">در حال بارگذاری...</div>
                  <div v-else-if="textContent" class="text-preview">{{ textContent }}</div>
                  <div v-else class="no-content">محتوای متنی موجود نیست</div>
                </div>
                
                <!-- PDF preview placeholder -->
                <div v-else-if="currentFile.type === 'application/pdf'" class="pdf-preview">
                  <div class="pdf-icon">📄</div>
                  <div class="pdf-text">برای مشاهده PDF، فایل را دانلود کنید</div>
                </div>
              </div>
            </template>
            
            <!-- Other file types preview -->
            <template v-else-if="currentFile && mediaKind === 'other'">
              <div class="other-preview">
                <div class="file-icon">{{ getFileIcon(currentFile) }}</div>
                <div class="file-info">
                  <div class="file-name">{{ currentFile.name }}</div>
                  <div class="file-size">{{ formatFileSize(currentFile.size) }}</div>
                  <div class="file-type">{{ currentFile.type || 'نوع فایل نامشخص' }}</div>
                </div>
                <div class="file-message">این نوع فایل قابل پیش‌نمایش نیست</div>
              </div>
            </template>
            
            <!-- No file selected -->
            <template v-else>
              <div class="hint">روی آیکن + برای انتخاب فایل کلیک کنید</div>
            </template>
          </div>

          <div class="thumbs" v-if="files.length">
            <button class="nav" @click="prev" :disabled="currentIndex===0"><LeftOutlined /></button>
            <div class="list">
              <div
                v-for="(f, i) in files"
                :key="f.name + i"
                class="thumb"
                :class="{ active: i === currentIndex }"
                @click="currentIndex = i"
                :title="f.name"
              >
                <div class="thumb-preview">
                  <img v-if="f.type.startsWith('image/')" :src="createThumbUrl(f)" @load="onThumbLoaded" />
                  <div v-else class="thumb-icon">{{ getFileIcon(f) }}</div>
                </div>
                <div class="thumb-name">{{ f.name }}</div>
              </div>
            </div>
            <button class="nav" @click="next" :disabled="currentIndex===files.length-1"><RightOutlined /></button>
          </div>
        </div>

        <div class="mbox-footer">
          <div class="left">تعداد فایل‌ها: {{ files.length }}</div>
          <div class="right">
            <button class="btn" @click="confirmAndClose">تأیید و بستن</button>
          </div>
        </div>
      </div>
  </dialog>
</template>

<style scoped>
.mbox-dialog{ 
  position: fixed; 
  inset: 0; 
  margin: auto; 
  padding: 0; 
  border: none; 
  background: transparent; 
  z-index: 1000;
  width: 60%;
  height: 80%;
}

.mbox-dialog::backdrop {
  background: var(--mbox-overlay-bg);
  backdrop-filter: blur(7px) saturate(1.5);
  -webkit-backdrop-filter: blur(12px) saturate(1.2);
}

/* Fallback for browsers that don't support backdrop-filter */
@supports not (backdrop-filter: blur(12px)) {
  .mbox-dialog::backdrop {
    background: rgba(78, 113, 255, 0.2);
  }
}

.mbox-modal{ 
  width: 100%; 
  height: 100%; 
  background: var(--mbox-panel-bg); 
  border: none; 
  border-radius: 16px; 
  display: flex; 
  flex-direction: column; 
  color: var(--mbox-text); 
  box-shadow: 0 20px 50px rgba(78, 113, 255, 0.1); 
  font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial; 
  font-weight: 500;
  overflow: hidden;
}
.mbox-header{ padding:10px 12px; display:flex; align-items:center; gap:8px; border-bottom:none }
.mbox-header .title{ font-weight: 600; color:#2E073F; font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial }
.mbox-header .actions{ margin-inline-start:auto; display:flex; gap:6px }
.icon-btn{ background:var(--mbox-canvas-bg); border:none; color:var(--mbox-text); height:32px; border-radius:8px; display:inline-flex; align-items:center; gap:6px; padding:0 8px; cursor:pointer; font-weight: 400; transition:all 0.2s ease; font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial }
.icon-btn:disabled{ opacity:.5; cursor:not-allowed }
.icon-btn:hover:not(:disabled){ background:var(--mbox-accent); color:var(--mbox-on-accent) }
.icon-btn.danger{ color:var(--mbox-danger); background:#fff5f5; font-weight: 500 }
.icon-btn.danger:hover:not(:disabled){ background:var(--mbox-danger); color:#ffffff }
.mbox-body{ display:flex; flex-direction:column; gap:8px; padding:10px }
.viewer{ position:relative; flex:1; min-height:350px; border:none; border-radius:12px; overflow:hidden; background:var(--mbox-canvas-bg); display:flex; align-items:center; justify-content:center; user-select:none }
.viewer img, .viewer video{ max-width:none; max-height:none; transform-origin:center center }
.viewer .centered{ position:absolute; top:50%; left:50%; }
.hint{ color:#2E073F; font-size:14px; opacity:0.7; font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial; font-weight: 300 }
.thumbs{ display:flex; align-items:center; gap:8px }
.thumbs .nav{ background:var(--mbox-canvas-bg); border:none; color:var(--mbox-text); width:32px; height:32px; border-radius:8px; display:inline-flex; align-items:center; justify-content:center; cursor:pointer; transition:all 0.2s ease; font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial; font-weight: 400 }
.thumbs .nav:disabled{ opacity:.5; cursor:not-allowed }
.thumbs .nav:hover:not(:disabled){ background:var(--mbox-accent); color:var(--mbox-on-accent) }
.thumbs .list{ display:flex; gap:6px; overflow:auto; padding:6px; border:none; border-radius:8px; flex:1; background:var(--mbox-canvas-bg) }
.thumbs .thumb{ padding:6px 10px; border:none; border-radius:8px; white-space:nowrap; max-width:240px; overflow:hidden; text-overflow:ellipsis; cursor:pointer; background:var(--mbox-panel-bg); display:flex; align-items:center; gap:8px; transition:all 0.2s ease; font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial; color:#2E073F; font-weight: 400 }
.thumbs .thumb .thumb-preview{ width:36px; height:28px; border:none; border-radius:8px; overflow:hidden; flex:none }
.thumbs .thumb .thumb-preview img{ width:100%; height:100%; object-fit:cover }
.thumbs .thumb .thumb-preview .thumb-icon{ width:100%; height:100%; display:flex; align-items:center; justify-content:center; font-size:18px }
.thumbs .thumb .thumb-name{ display:block; overflow:hidden; text-overflow:ellipsis }
.thumbs .thumb.active{ background:var(--mbox-accent); color:var(--mbox-on-accent); font-weight: 500 }
.thumbs .thumb:hover:not(.active){ background:var(--mbox-canvas-bg) }

/* Preview styles for different file types */
.audio-preview, .document-preview, .other-preview{ display:flex; flex-direction:column; align-items:center; justify-content:center; gap:16px; padding:20px; text-align:center }
.audio-icon, .document-icon, .file-icon{ font-size:48px; margin-bottom:8px }
.audio-player{ width:100%; max-width:400px }
.file-info{ display:flex; flex-direction:column; gap:4px; color:var(--mbox-text) }
.file-name{ font-weight:600; font-size:16px; word-break:break-all }
.file-size{ font-size:14px; opacity:0.8 }
.file-type{ font-size:12px; opacity:0.6; font-family:monospace }
.text-content{ width:100%; max-width:600px; max-height:300px; overflow-y:auto; background:var(--mbox-canvas-bg); border:1px solid var(--mbox-border); border-radius:8px; padding:12px; text-align:right; direction:rtl }
.text-preview{ white-space:pre-wrap; font-family:monospace; font-size:12px; line-height:1.4 }
.loading{ color:var(--mbox-accent); font-style:italic }
.no-content{ color:var(--mbox-text); opacity:0.6; font-style:italic }
.pdf-preview{ display:flex; flex-direction:column; align-items:center; gap:12px }
.pdf-icon{ font-size:64px }
.pdf-text{ color:var(--mbox-text); opacity:0.8 }
.file-message{ color:var(--mbox-text); opacity:0.6; font-style:italic; margin-top:8px }
.mbox-footer{ display:flex; align-items:center; justify-content:space-between; gap:8px; padding:8px 10px; border-top:none }
.btn{ background:var(--mbox-accent); color:var(--mbox-on-accent); border:none; padding:10px 16px; border-radius:8px; cursor:pointer; font-weight: 600; transition:all 0.2s ease; font-family: 'Pelak', 'Kalameh', IRANYekanX, Vazirmatn, Inter, system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial }
.btn:hover{ background:#1e40af }
</style>

