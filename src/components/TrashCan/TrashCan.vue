<template>
  <div
    class="window"
    ref="windowRef"
    :class="{ maximized: isMaximized }"
    :style="windowStyle"
    @mousedown="$emit('bring-to-front')"
  >
    <div class="title-bar" ref="titleBarRef">
      <span>回收站</span>
      <div class="window-buttons">
        <button @mousedown.stop @click="toggleMaximize">口</button>
        <button @mousedown.stop @click="$emit('close')">X</button>
      </div>
    </div>

    <div class="content">
      <p>回收站是空的</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

const emit = defineEmits(['close', 'bring-to-front'])
const props = defineProps({
  ieLeft: Number,
  ieTop: Number,
  zIndex: Number
})

// ----------------------
// 窗口位置和大小
// ----------------------
const windowRef = ref(null)
const titleBarRef = ref(null)
const width = ref(400)
const height = ref(300)
const initialLeft = ref((props.ieLeft || 100) + 20)
const initialTop = ref((props.ieTop || 80) + 20)

let isDragging = false
let offsetX = 0
let offsetY = 0

// 最大化状态
const isMaximized = ref(false)
const prevState = ref({ left: 0, top: 0, width: 0, height: 0 })

// ----------------------
// 计算窗口样式
// ----------------------
const windowStyle = computed(() => ({
  left: isMaximized.value ? '0px' : initialLeft.value + 'px',
  top: isMaximized.value ? '0px' : initialTop.value + 'px',
  width: isMaximized.value ? '100%' : width.value + 'px',
  height: isMaximized.value ? '100%' : height.value + 'px',
  zIndex: props.zIndex || 10
}))

// ----------------------
// 拖动逻辑
// ----------------------
const onMouseDownDrag = (e) => {
  emit('bring-to-front')
  if (!windowRef.value || isMaximized.value) return
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

// ----------------------
// 最大化/恢复
// ----------------------
const toggleMaximize = () => {
  emit('bring-to-front')
  if (!isMaximized.value) {
    prevState.value = {
      left: initialLeft.value,
      top: initialTop.value,
      width: width.value,
      height: height.value
    }
    isMaximized.value = true
  } else {
    initialLeft.value = prevState.value.left
    initialTop.value = prevState.value.top
    width.value = prevState.value.width
    height.value = prevState.value.height
    isMaximized.value = false
  }
}

onMounted(() => {
  titleBarRef.value?.addEventListener('mousedown', onMouseDownDrag)
})

onBeforeUnmount(() => {
  titleBarRef.value?.removeEventListener('mousedown', onMouseDownDrag)
})
</script>

<style scoped>
.window {
  position: absolute;
  background: #c0c0c0;
  border: 2px solid #fff;
  border-right-color: #808080;
  border-bottom-color: #808080;
  box-shadow: inset -1px -1px 0 #fff, inset 1px 1px 0 #000;
  user-select: none;
  display: flex;
  flex-direction: column;
  font-family: Tahoma, sans-serif;
  font-size: 12px;
}

.title-bar {
  background: #000080;
  color: white;
  padding: 2px 5px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  cursor: move;
  font-weight: bold;
  text-shadow: 1px 1px #00000080;
  border-bottom: 2px solid #fff;
  border-right: 2px solid #808080;
}

.window-buttons {
  display: flex;
}

.window-buttons button {
  border: 2px solid #fff;
  border-right-color: #808080;
  border-bottom-color: #808080;
  margin-left: 2px;
  font-size: 12px;
  cursor: pointer;
  width: 18px;
  height: 18px;
  padding: 0;
  line-height: 14px;
  background: #c0c0c0;
  font-weight: bold;
}

.window-buttons button:hover {
  background: #808080;
  border: 2px solid #fff;
  border-right-color: #000;
  border-bottom-color: #000;
}

.content {
  padding: 10px;
  background: #ffffff56;
  border-top: 2px solid #808080;
  border-left: 2px solid #fff;
  overflow: auto;
  flex: 1;
}
</style>
