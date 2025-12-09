<template>
  <div
  class="window"
  ref="windowRef"
  :style="windowStyle"
  @mousedown="$emit('bring-to-front')"
>
   <div class="title-bar" ref="titleBarRef">
  Chat
  <button @click="$emit('close')">X</button>
</div>

    <div class="content">
      <div class="messages">
        <div v-for="(msg, index) in messages" :key="index">{{ msg }}</div>
      </div>

      <input v-model="inputText" @keyup.enter="send" placeholder="输入消息..." />
      <button @click="send">发送</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount, toRef } from 'vue';

const emit = defineEmits(['close', 'send', 'bring-to-front']);
const props = defineProps({
  messages: Array,
  zIndex: Number,
   activeContact: Object   // 新增
});

// 输入框数据
const inputText = ref('');
const messages = toRef(props, 'messages');

function send() {
  if (!inputText.value.trim()) return;
  emit('send', inputText.value.trim());
  inputText.value = '';
}

// ----------------------
// 拖动与位置
// ----------------------
const windowRef = ref(null);
const titleBarRef = ref(null);

const initialLeft = ref(150);
const initialTop = ref(150);
const width = ref(300);
const height = ref(400);

let isDragging = false;
let offsetX = 0;
let offsetY = 0;

const windowStyle = computed(() => ({
  left: initialLeft.value + 'px',
  top: initialTop.value + 'px',
  width: width.value + 'px',
  height: height.value + 'px',
  zIndex: props.zIndex || 10
}));

const onMouseDownDrag = (e) => {
  emit('bring-to-front'); // 通知父组件置顶
  if (!windowRef.value) return;

  e.preventDefault();
  isDragging = true;

  offsetX = e.clientX - windowRef.value.offsetLeft;
  offsetY = e.clientY - windowRef.value.offsetTop;

  document.addEventListener('mousemove', onMouseMove);
  document.addEventListener('mouseup', onMouseUp);
};

const onMouseMove = (e) => {
  if (!isDragging || !windowRef.value) return;

  let left = e.clientX - offsetX;
  let top = e.clientY - offsetY;

  const parent = windowRef.value.parentElement;
  if (parent) {
    const pw = parent.clientWidth;
    const ph = parent.clientHeight;
    const rw = windowRef.value.offsetWidth;
    const rh = windowRef.value.offsetHeight;

    left = Math.max(0, Math.min(left, pw - rw));
    top = Math.max(0, Math.min(top, ph - rh));
  }

  initialLeft.value = left;
  initialTop.value = top;
};

const onMouseUp = () => {
  isDragging = false;
  document.removeEventListener('mousemove', onMouseMove);
  document.removeEventListener('mouseup', onMouseUp);
};

onMounted(() => {
  titleBarRef.value?.addEventListener('mousedown', onMouseDownDrag);
});

onBeforeUnmount(() => {
  titleBarRef.value?.removeEventListener('mousedown', onMouseDownDrag);
});
</script>


<style scoped>
.window { width: 300px; height: 400px; background: white; border: 1px solid #999; position: absolute; top: 150px; left: 150px; }
.title-bar { background: #00a; color: white; padding: 5px; display: flex; justify-content: space-between; }
.messages { height: 250px; overflow-y: auto; border-bottom: 1px solid #ccc; padding: 5px; }
.links { margin: 5px 0; }
.link { display: inline-block; margin-right: 10px; color: blue; cursor: pointer; text-decoration: underline; }
</style>
