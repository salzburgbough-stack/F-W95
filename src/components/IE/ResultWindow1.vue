<template>
  <div class="window" ref="windowRef" :class="{ maximized: isMaximized }" :style="windowStyle"
    @mousedown="bringToFront">
    <div class="title-bar" ref="titleBarRef">
      <span>搜索结果</span>
      <div class="window-buttons">
        <button @click="toggleMaximize">口</button>
        <button @click="$emit('close')">X</button>
      </div>
    </div>

    <div class="content">
      <h3>您搜索了：{{ keyword }}</h3>
      <p>网页没有找到相关内容，请 <span class="return-link" @click="handleReturn">这里返回</span> 或关闭窗口。</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

const props = defineProps({
  keyword: String,
  zIndex: Number,
  parentWidth: Number,
  parentHeight: Number
})
const emit = defineEmits(['close', 'bring-to-front', 'open-browser'])
const handleReturn = () => {
  emit('close')          // 关闭当前窗口
  emit('open-browser')   // 通知外层打开浏览器窗口
}

const windowRef = ref(null)
const titleBarRef = ref(null)

const width = ref(400)
const height = ref(300)

// 初始位置比 IE 窗口略微偏移 10~20px
const initialLeft = ref((props.ieLeft || 100) + 30 + Math.random() * 50)
const initialTop = ref((props.ieTop || 80) + 30 + Math.random() * 50)
let isDragging = false
let offsetX = 0
let offsetY = 0

const isMaximized = ref(false)
const prevState = ref({ left: 0, top: 0, width: 0, height: 0 })

const windowStyle = computed(() => ({
  left: isMaximized.value ? '0px' : initialLeft.value + 'px',
  top: isMaximized.value ? '0px' : initialTop.value + 'px',
  width: isMaximized.value ? '100%' : width.value + 'px',
  height: isMaximized.value ? '100%' : height.value + 'px',
  zIndex: props.zIndex || 10
}))

// 拖动逻辑
const onMouseDown = (e) => {
  if (isMaximized.value) return
  isDragging = true
  offsetX = e.clientX - initialLeft.value
  offsetY = e.clientY - initialTop.value
  bringToFront()
  document.addEventListener('mousemove', onMouseMove)
  document.addEventListener('mouseup', onMouseUp)
}

const onMouseMove = (e) => {
  if (!isDragging) return
  let left = e.clientX - offsetX
  let top = e.clientY - offsetY

  // 限制在父容器内
  if (props.parentWidth && props.parentHeight) {
    left = Math.max(0, Math.min(left, props.parentWidth - width.value))
    top = Math.max(0, Math.min(top, props.parentHeight - height.value))
  }

  initialLeft.value = left
  initialTop.value = top
}

const onMouseUp = () => {
  isDragging = false
  document.removeEventListener('mousemove', onMouseMove)
  document.removeEventListener('mouseup', onMouseUp)
}

// 最大化切换
const toggleMaximize = () => {
  bringToFront()
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

const bringToFront = () => {
  emit('bring-to-front')
}

onMounted(() => {
  titleBarRef.value?.addEventListener('mousedown', onMouseDown)
})

onBeforeUnmount(() => {
  titleBarRef.value?.removeEventListener('mousedown', onMouseDown)
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
  padding: 8px;
  background: #ffffff56;
  border-top: 2px solid #808080;
  border-left: 2px solid #fff;
  overflow: auto;
  flex: 1;
}
.return-link {
  color: blue;
  text-decoration: underline;
  cursor: pointer;
}

</style>
