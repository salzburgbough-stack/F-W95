<template>
  <div class="window" ref="windowRef" :style="windowStyle">
    <div class="title-bar" ref="titleBarRef">
      MIMES 95
      <button @click="$emit('close')">X</button>
    </div>

    <div class="content">

      <!-- 分组 -->
      <div class="group" v-for="(g, i) in groups" :key="g.name">

        <!-- 点击分组：展开 / 收起 -->
        <div class="group-title" @click="toggle(i)">
          <span class="arrow" :class="{ open: g.open }">▶</span>
          <!-- 分組名稱 + 在線/總人數 -->
          <template v-if="g.name === '朋友'">
            {{ g.name }}（{{g.contacts.filter(c => c.online).length}} / {{ g.total }}）
          </template>
          <template v-else>
            {{ g.name }}（0 / {{ g.total }}） <!-- 固定在線人數為 0 -->
          </template>
        </div>

        <!-- 联系人列表（可折叠） -->
        <div v-show="g.open">
          <div class="contact" v-for="c in g.contacts" :key="c.id" @dblclick="choose(c)">
            {{ c.name }}
          </div>
        </div>

      </div>
    </div>
    <div class="version">v1.6.0</div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

const emit = defineEmits(['close', 'bring-to-front', 'open-contact'])
const props = defineProps({ zIndex: Number })

// ----------------------------
// 联系人分组（增加 open: true 控制展开）
// ----------------------------
const groups = ref([
  {
    name: "家人",
    total: 2,
    open: true,
    contacts: [
      { id: 1, name: "爸" },
      { id: 2, name: "妈" }   // ← 新增
    ]
  },
  {
    name: "同事",
    total: 0,
    open: true,
    contacts: []
  },
  {
    name: "朋友",
    total: 1,
    open: true,
    contacts: [
      { id: 2, name: "用户89757" }
    ]
  }
])

function toggle(index) {
  groups.value[index].open = !groups.value[index].open
}

function choose(c) {
  emit("open-contact", c)
}

// ----------------------
// 拖动逻辑（不变）
// ----------------------
const windowRef = ref(null)
const titleBarRef = ref(null)

const left = ref(120)
const top = ref(20)
const width = ref(170)
const height = ref(300)

let isDragging = false
let offsetX = 0
let offsetY = 0

const windowStyle = computed(() => ({
  position: 'absolute',
  left: left.value + 'px',
  top: top.value + 'px',
  width: width.value + 'px',
  height: height.value + 'px',
  zIndex: props.zIndex
}))

const onMouseDownDrag = (e) => {
  emit('bring-to-front')
  isDragging = true
  offsetX = e.clientX - left.value
  offsetY = e.clientY - top.value
  document.addEventListener('mousemove', onMouseMove)
  document.addEventListener('mouseup', onMouseUp)
}

const onMouseMove = (e) => {
  if (!isDragging) return

  // 计算新的位置
  let newLeft = e.clientX - offsetX
  let newTop = e.clientY - offsetY

  // 获取父容器大小，限制窗口不超出
  const parent = windowRef.value?.parentElement
  if (parent) {
    const maxLeft = parent.clientWidth - windowRef.value.offsetWidth
    const maxTop = parent.clientHeight - windowRef.value.offsetHeight
    newLeft = Math.max(0, Math.min(newLeft, maxLeft))
    newTop = Math.max(0, Math.min(newTop, maxTop))
  }

  left.value = newLeft
  top.value = newTop
}

const onMouseUp = () => {
  isDragging = false
  document.removeEventListener('mousemove', onMouseMove)
  document.removeEventListener('mouseup', onMouseUp)
}

onMounted(() => {
  titleBarRef.value.addEventListener('mousedown', onMouseDownDrag)
})
onBeforeUnmount(() => {
  titleBarRef.value.removeEventListener('mousedown', onMouseDownDrag)
})
</script>

<style scoped>
.window {
  background: #ffffff; /* 淡灰底，复古感 */
  overflow: hidden;
  position: absolute;
  font-family: 'Tahoma', 'Arial', sans-serif;
  border: 2px solid #d0d0d0;

}

.title-bar {
  background: linear-gradient(145deg, #222, #444); /* 渐变黑色复古风 */
  color: #fff;
  padding: 4px 8px;
  font-weight: bold;
  font-size: 12px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  cursor: move;
  border-bottom: 2px solid #a00; /* 红色底线保留 */
  text-shadow: 1px 1px #0008;
  
}

button {
  background: linear-gradient(145deg, #aa4444, #770000); /* 暗红渐变，更复古 */
  border: 1px solid #550000;
  color: #fff;
  font-weight: bold;
  font-size: 10px;
  width: 20px;
  height: 20px;
  cursor: pointer;
  border-radius: 50%; /* 圆形 */
  display: flex;
  justify-content: center;
  align-items: center;
  box-shadow: 1px 1px 0 #fff inset, -1px -1px 0 #550000 inset; /* 浮雕内凹 */
  transition: all 0.2s ease;
  padding: 0;
}

button:hover {
  background: linear-gradient(145deg, #bb5555, #882222); /* 悬停稍亮但仍复古 */
}

button:active {
  box-shadow: inset 2px 2px 2px #550000;
}

.content {
  padding: 6px;
  height: calc(100% - 42px);
  font-size: 12px;
  color: #222;
  display: flex;
  flex-direction: column;
  border-top: 1px solid #888; /* 轻微边框，柔和黑色 */
}

.group {
  margin-bottom: -5px;
  padding-bottom: 10px;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.group-title {
  font-weight: bold;
  font-size: 12px;
  color: #111;
  margin-bottom: 4px;
  cursor: pointer;
  user-select: none;
  padding: 2px 4px;
  text-align: left;
}

.group-title:hover {
  color: #a00; /* 悬停红色 */
  text-decoration: underline;
}

.contact {
  padding: 4px 8px;
  margin: 0;
  cursor: pointer;
  border: none;
  background: transparent;
  font-size: 12px;
  color: #111;
  text-align: left;
  width: 100%;
  user-select: none;
  transition: color 0.2s ease;
}

.contact:hover {
  color: #a00; /* 悬停红色文字 */
}

.arrow {
  display: inline-block;
  transition: transform 0.2s ease;
  margin-right: 4px;
  color: #222;
}

.arrow.open {
  transform: rotate(90deg);
}
.version {
  position: absolute;
  bottom: 4px;
  right: 6px;
  font-size: 10px;
  color: #888;           /* 灰色 */
  user-select: none;
  pointer-events: none;   /* 不影响点击 */
}

</style>
