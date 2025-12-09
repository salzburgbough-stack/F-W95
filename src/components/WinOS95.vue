<template>
  <div id="desktop">
    <!-- 桌面图标 -->
    <div id="icons">
      <div v-for="icon in icons" :key="icon.id" class="icon" @dblclick="openProgram(icon.name)">
        <img :src="icon.img || '/public/ProgramIcon/w98_msagent.ico'" alt="icon" />
        <span>{{ icon.name }}</span>
      </div>
    </div>

    <!-- 打开的程序窗口 -->
    <component v-for="program in programs" :is="program.component" v-show="program.isOpen" :key="program.name"
      v-bind="program.props" @mousedown.native="bringToFront(program)" @close="closeProgram(program.name)"
      @search="handleSearch" @send="handleChatInput(program.name, $event)" @input-change="handleIEInput"
      class="component-window" />
    <!-- 动态窗口（例如 IE 搜索结果窗口） -->
    <component v-for="win in dynamicWindows" :is="win.component" :key="win.id" v-bind="win.props"
      @mousedown.native="bringToFront(win)" @close="closeDynamicWindow(win.id)" class="component-window" />

    <!-- 任务栏 -->
    <div id="taskbar">
      <div id="start-btn" @click="toggleStartMenu">开始</div>
      <div style="display:flex; align-items:center;">
        <div id="sound-icon" @click="toggleSound">
          <img :src="isMuted ? '/public/ProgramIcon/display.png' : '/public/ProgramIcon/play.png'" />
        </div>
        <div id="time-display">{{ time }}</div>
      </div>
    </div>

    <!-- 开始菜单 -->
    <div id="start-menu" v-show="showStartMenu">
      <div @click="systemAction('Shutdown')">关机</div>
      <div @click="systemAction('Restart')">重启</div>
      <div @click="systemAction('Logout')">注销</div>

    </div>

    <!-- CRT 扫描线 -->
    <div id="crt-stripe"></div>
  </div>
</template>

<script setup>
import { reactive, ref, onMounted } from 'vue';
import IE from './IE/IE.vue';
import MIMES95 from './Chat/MIMES95.vue';
import Documents from './Documents/Document.vue';
import MyComputer from './MyComputer/MyComputer.vue';
import TrashCan from './TrashCan/TrashCan.vue';
import ResultWindow1 from './IE/ResultWindow1.vue';

//声音控制
// 音量状态
const isMuted = ref(false);
// 切换静音
function toggleSound() {
  isMuted.value = !isMuted.value;
  // 控制页面所有音频/视频
  const audios = document.querySelectorAll('audio, video');
  audios.forEach(media => {
    media.muted = isMuted.value;
  });
}


// 全局 Z 轴
const topZIndex = ref(10)
//IE 新搜索窗口
const dynamicWindows = reactive([]);
// 桌面图标
const icons = reactive([
  { id: 1, name: '我的电脑', img: '/public/ProgramIcon/icons8-windows-95-我的电脑-48.png' },
  { id: 2, name: '我的文档', img: '/public/ProgramIcon/icons8-windows-95-我的文档-48.png' },
  { id: 3, name: '回收站', img: '/public/ProgramIcon/icons8-windows-95-回收站-48.ico' },
  { id: 4, name: 'IE 浏览器', img: '/public/ProgramIcon/icons8-windows-95-浏览器-48.png' },
  { id: 5, name: 'MIMES 95', img: '/public/ProgramIcon/w98_friend.ico' },
]);

// 程序窗口
const programs = reactive([
  { name: 'IE 浏览器', component: IE, isOpen: false, props: { zIndex: topZIndex.value } },
  { name: 'MIMES 95', component: MIMES95, isOpen: false, props: reactive({ messages: [], zIndex: topZIndex.value }) },
  { name: '我的文档', component: Documents, isOpen: false, props: { documents: [], zIndex: topZIndex.value } },
  { name: '我的电脑', component: MyComputer, isOpen: false, props: { zIndex: topZIndex.value } },
  { name: '回收站', component: TrashCan, isOpen: false, props: { zIndex: topZIndex.value } },
]);
// 用于记录已经触发过的关键词
const triggeredKeywords = reactive(new Set());

// 点击或拖动窗口置顶
function bringToFront(window) {
  topZIndex.value++
  window.props = window.props || {}
  window.props.zIndex = topZIndex.value
}
// 打开动态窗口
function openDynamicWindow(component, props = {}) {
  const win = {
    id: Date.now() + Math.random(),
    component,
    props: { ...props, zIndex: topZIndex.value + 1 } // 初始比topZIndex高
  }
  topZIndex.value++
  dynamicWindows.push(win)
  bringToFront(win)
}
function closeDynamicWindow(id) {
  const index = dynamicWindows.findIndex(win => win.id === id);
  if (index !== -1) dynamicWindows.splice(index, 1);
}

// 打开程序
function openProgram(name) {
  const program = programs.find(p => p.name === name);
  if (program) program.isOpen = true;
}

// 关闭程序
function closeProgram(name) {
  const program = programs.find(p => p.name === name);
  if (program) program.isOpen = false;
}

// 获取 Chat 程序对象
function getChatProgram() {
  return programs.find(p => p.name === 'MIMES 95');
}

function handleChatInput(programName, message) {
  const program = programs.find(p => p.name === programName)
  if (!program) return
  program.props.messages.push('你：'+ message)
}


// IE 输入事件处理
function handleIEInput(value) {
}

// IE 搜索事件
function handleSearch(keyword) {
  if (!keyword) return;

  // ① 弹出新窗口
  openDynamicWindow(ResultWindow1, { keyword });

  // ② 原有功能仍然工作
  const chat = getChatProgram();
  if (!chat) return;

  if (keyword === '777') {
    chat.props.messages.push('系统: 000');
  } else if (keyword === '222') {
    chat.props.messages.push('系统: 123456');
  }
}


// 我的电脑点击盘符事件
function handleDiskClick(diskName) {
  const chat = getChatProgram();
  if (!chat) return;
  // 不再自动打开 Chat 窗口
  chat.props.messages.push('系统: 现在不是娱乐的时候');
}


// 开始菜单
const showStartMenu = ref(false);
function toggleStartMenu() {
  showStartMenu.value = !showStartMenu.value;
}
function systemAction(action) {
  alert(`执行系统操作: ${action}`);
}

// 时间显示
const time = ref('');
function updateTime() {
  const now = new Date();
  let h = now.getHours();
  const ampm = h >= 12 ? 'PM' : 'AM';
  h = h % 12 || 12;
  const m = now.getMinutes().toString().padStart(2, '0');
  time.value = `${h}:${m} ${ampm}`;
}

onMounted(() => {
  updateTime();
  setInterval(updateTime, 1000);

  // CRT扫描线动画
  const stripe = document.getElementById('crt-stripe');
  let pos = -100;
  function animate() {
    pos += 0.5;
    if (pos > 100) pos = -100;
    stripe.style.top = pos + '%';
    requestAnimationFrame(animate);
  }
  animate();

    // 页面加载时确保音频未静音
  const audios = document.querySelectorAll('audio, video');
  audios.forEach(media => media.muted = false);
});
</script>

<style scoped>
/* 浏览器全屏容器，居中显示复古桌面 */
body,
html {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  background: #222;
  display: flex;
  justify-content: center;
  align-items: center;
  font-family: "MS Sans Serif", Arial, sans-serif;
}

/* 桌面 */
#desktop {
  position: relative;
  top: 12px;
  left: 2px;
  width: 770px;
  height: 600px;
  background-color: #236e5c15;
  border: 8px solid #000;
  box-shadow: 0 0 30px rgba(0, 0, 0, 0.7), inset 0 0 10px rgba(255, 255, 255, 0.05);
  overflow: hidden;
  display: block;
}

/* 桌面图标 */
#icons {
  position: absolute;
  top: 52px;
  left: 20px;
  display: grid;
  grid-template-columns: repeat(auto-fill, 48px);
  grid-auto-rows: 64px;
  gap: 30px 20px;
  z-index: 2;
}

.icon {
  width: 70px;
  height: 48px;
  text-align: center;
  cursor: pointer;
}

.icon img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.icon span {
  display: block;
  font-size: 12px;
  color: white;
  text-align: center;
  margin-top: 2px;
  text-shadow: 1px 1px #000;
}

/* CRT 扫描线 */
#crt-stripe {
  position: absolute;
  mix-blend-mode: multiply;
  top: -100%;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  background-image: repeating-linear-gradient(180deg,
      rgba(255, 255, 255, 0.05) 0,
      rgba(255, 255, 255, 0.05) 2px,
      rgba(0, 0, 0, 0.05) 2px,
      rgba(0, 0, 0, 0.05) 4px);
  opacity: 0.15;
  z-index: 50;
}

/* 任务栏 */
#taskbar {
  position: absolute;
  bottom: 5px;
  width: 737px;
  left: 10px;
  right: unset;
  height: 22px;
  background: #c0c0c0a1;
  border-top: 1px solid #888;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 5px;
  z-index: 2;
  font-size: 12px;
}

#start-btn {
  /*开始按钮 */
  width: 50px;
  height: 20px;
  background: #c0c0c08a;
  border: 1px outset #ffffffab;
  text-align: center;
  line-height: 20px;
  cursor: pointer;
  user-select: none;
}

#sound-icon {
  width: 20px;              
  height: 20px;             
  display: flex;             
  align-items: center;       
  justify-content: center;  
  margin-right: 5px;         
  cursor: pointer;           
}

#sound-icon img {
  width: 14px;               
  height: 14px;
  image-rendering: pixelated; 
}


#time-display {
  min-width: 60px;
  text-align: center;
}

/* 开始菜单的打开栏 */
#start-menu {
  position: absolute;
  bottom: 30px;
  left: 5px;
  width: 120px;
  background: #C0C0C0;
  border: 2px solid #000;
  z-index: 3;
  font-size: 12px;
}

#start-menu div {
  padding: 4px;
  cursor: pointer;
  border-bottom: 1px solid #888;
}

#start-menu div:last-child {
  border-bottom: none;
}

.component-window {
  position: absolute;
  overflow: visible;
}
</style>
