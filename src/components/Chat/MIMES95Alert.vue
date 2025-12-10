<template>
  <div
    class="window"
    ref="windowRef"
    :style="windowStyle"
    @mousedown="$emit('bring-to-front')"
  >
    <div class="title-bar" ref="titleBarRef">
      提示
      <button @mousedown.stop @click="$emit('close')">X</button>
    </div>
    <div class="content">
      <p>该用户未开通 MIMES 95 聊天功能</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

const emit = defineEmits(['close', 'bring-to-front'])
const props = defineProps({
  zIndex: Number,
  left: { type: Number, default: 200 },
  top: { type: Number, default: 200 },
  width: { type: Number, default: 300 },
  height: { type: Number, default: 150 },
})

const windowRef = ref(null)
const titleBarRef = ref(null)

let isDragging = false
let offsetX = 0
let offsetY = 0

const initialLeft = ref(props.left)
const initialTop = ref(props.top)

const onMouseDownDrag = (e) => {
  emit('bring-to-front')
  if (!windowRef.value) return
  e.preventDefault()
  isDragging = true
  offsetX = e.clientX - windowRef.value.offsetLeft
  offsetY = e.clientY - windowRef.value.offsetTop
  document.addEventListener('mousemove', onMouseMove)
  document.addEventListener('mouseup', onMouseUp)
}

const onMouseMove = (e) => {
  if (!isDragging || !windowRef.value) return
  let left = e.clientX - offsetX
  let top = e.clientY - offsetY
  const parent = windowRef.value.parentElement
  if (parent) {
    const pw = parent.clientWidth
    const ph = parent.clientHeight
    const rw = windowRef.value.offsetWidth
    const rh = windowRef.value.offsetHeight
    left = Math.max(0, Math.min(left, pw - rw))
    top = Math.max(0, Math.min(top, ph - rh))
  }
  initialLeft.value = left
  initialTop.value = top
}

const onMouseUp = () => {
  isDragging = false
  document.removeEventListener('mousemove', onMouseMove)
  document.removeEventListener('mouseup', onMouseUp)
}

onMounted(() => {
  titleBarRef.value?.addEventListener('mousedown', onMouseDownDrag)
})

onBeforeUnmount(() => {
  titleBarRef.value?.removeEventListener('mousedown', onMouseDownDrag)
})

const windowStyle = computed(() => ({
  position: 'absolute',
  left: initialLeft.value + 'px',
  top: initialTop.value + 'px',
  width: props.width + 'px',
  height: props.height + 'px',
  zIndex: props.zIndex,
}))
</script>

<style scoped>
.window {
  background: #ffffff; 
  overflow: hidden;
  position: absolute;
  font-family: 'Tahoma', 'Arial', sans-serif;
  border: 2px solid #d0d0d0;
  box-shadow: inset -2px -2px 0 #fff, inset 2px 2px 0 #888;
  display: flex;
  flex-direction: column;
}

.title-bar {
  background: linear-gradient(145deg, #222, #444);
  color: #fff;
  padding: 4px 8px;
  font-weight: bold;
  font-size: 12px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  cursor: move;
  border-bottom: 2px solid #a00;
  text-shadow: 1px 1px #0008;
}

.title-bar button {
  background: linear-gradient(145deg, #aa4444, #770000);
  border: 1px solid #550000;
  color: #fff;
  font-weight: bold;
  font-size: 10px;
  width: 20px;
  height: 20px;
  cursor: pointer;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  box-shadow: 1px 1px 0 #fff inset, -1px -1px 0 #550000 inset;
  transition: all 0.2s ease;
  padding: 0;
}

.title-bar button:hover {
  background: linear-gradient(145deg, #bb5555, #882222);
}

.title-bar button:active {
  box-shadow: inset 2px 2px 2px #550000;
}

.content {
  padding: 10px;
  background: linear-gradient(to bottom, #eee, #ccc);
  font-size: 12px;
  color: #222;
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  border-top: 1px solid #888;
}
</style>
