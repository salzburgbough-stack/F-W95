<script setup>
import { ref, onMounted } from 'vue';
import WinOS95 from './components/WinOS95.vue';

const showLogin = ref(true);
const messageLines = [
  // "[System Notice] -----------------------------------------",
  // "  未来科技 Corp.",
  // "  Produced & Distributed by",
  // ">Connecting to the supercomputer “Pixel Loop”...",
  // "> INITIALIZING NARRATIVE ENGINE...",
  // "---------------------------------------------------------",
  // "",
  // "> Target Node: System Update Files",
  // "> Synchronizing with technical servers...",
  // "> CHECKING MENTAL STABILITY... OK",
  // "> SCANNING FOR UNKNOWN ENTITIES... MINOR ANOMALIES DETECTED",
  // "Update Progress: ",
  // " *█████████████████████████████",
  // "",
  // "",
  // "> WARNING: ",
  // "> 本游戏含有轻微惊悚元素，可能对部分玩家造成心理或生理负担。",
  // "> 无视警告继续将被视为充分了解风险后的自主选择。",
  // "> 情节与角色皆为虚构。若与现实雷同，纯属巧合，请仔细甄别。",
  // "",
  // "> 你也许还没有准备好去探究那些深处的秘密……",
  // "> 每一次冒险都隐藏着未知的挑战，",
  // "> 有些门扉暂时还不属于你。",
  // "> 当你准备好再次踏上旅程时，",
  // "> 正义与真相依然在角落里安静等待。",
  // "",
  // "> 如果此刻你想停下。",
  // "> 请点击右上角退出。",
  // "",
  // "",
  // "",
];

const displayedMessage = ref('');
const typingFinished = ref(false);

const typeMessage = async () => {
  displayedMessage.value = '';
  for (let line of messageLines) {
    for (let char of line) {
      displayedMessage.value += char;
      await new Promise(resolve => setTimeout(resolve, 50));
    }
    displayedMessage.value += '\n';
    await new Promise(resolve => setTimeout(resolve, 300));
  }
  typingFinished.value = true;
};

onMounted(() => {
  typeMessage();
});

const startGame = () => {
  showLogin.value = false;
};
</script>

<template>
  <!-- 登录遮罩 -->
  <div v-if="showLogin" class="login-overlay">
    <div class="login-box">
      <pre class="login-message" :class="{ 'fade-out': typingFinished }">{{ displayedMessage }}</pre>

      <!-- 开始按钮 -->
      <div v-if="typingFinished" class="start-button" @click="startGame">
        >开始
      </div>
    </div>
  </div>

  <!-- 游戏主界面 -->
  <WinOS95 v-else msg="Vite + Vue" />
</template>

<style>
body,
html {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  background-image: url('./assets/ImageSource/backgrand7.png');
  background-size: 1880px 1060px;
  background-position: center;
  background-repeat: no-repeat;
  font-family: 'Arial', sans-serif;
  /* background-attachment: fixed; */
}

.login-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: transparent;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}

.login-message {
  font-size: 18px;
  color: #00ffff;
  text-align: left;
  line-height: 1.6;
  text-shadow:
    0 0 3px #00ffff,
    0 0 6px #00ffff,
    0 0 9px #00ffff;
  max-width: 600px;
  margin-left: auto;
  margin-right: auto;
  white-space: pre-wrap;
  word-wrap: break-word;
  height: 520px;
  width: 620px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  opacity: 1;
  transition: opacity 0.5s ease;
}

/* 开始按钮样式 */
.start-button {
  font-size: 18px;
  color: #00ffff;
  text-align: center;
  cursor: pointer;
  text-shadow:
    0 0 3px #00ffff,
    0 0 6px #00ffff,
    0 0 9px #00ffff;
  user-select: none;
  position: relative;
  /* 新增 */
  top: -50px;
  /* 往上移动 */
  left: -270px;
  /* 往左移动*/
}

.start-button:hover {
  color: #ffffff;
  text-shadow:
    0 0 5px #00ffff,
    0 0 10px #00ffff,
    0 0 15px #00ffff;
}
</style>
