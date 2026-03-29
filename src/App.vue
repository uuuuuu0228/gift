<template>
  <div class="app-shell">
    <!-- 1. 静态背景系统 -->
    <div class="background-system">
      <div class="bg-layer" :style="{ backgroundImage: `url(${晚霞})` }"></div>
      <div class="bg-mask-static"></div>
    </div>

    <!-- 2. 主舞台区 -->
    <div class="main-stage">
      <transition name="scene-change" mode="out-in">

        <!-- [场景：首页] -->
        <section v-if="viewMode === 'home'" key="home" class="scene-container">
          <div class="glass-card entrance-card">
            <div class="avatar-ripple">
              <img :src="头像" class="main-avatar" />
            </div>
            <h1 class="welcome-text">20岁·生日特别企划</h1>
            <p class="welcome-sub">嘿，你有一些未读消息</p>
            <button class="start-btn" @click="startChat">点击查看</button>
          </div>
        </section>

        <!-- [场景：IM聊天室] -->
        <section v-else-if="viewMode === 'chat'" key="chat" class="scene-container">
          <div class="chat-window gpu-accelerate">
            <header class="chat-header">
              <div :class="['nickname', { 'deactivated-text': !headerState.isOnline }]">
                {{ headerState.nickname }}
              </div>
              <div class="status-row">
                <span :class="['status-text', headerState.isOnline ? 'online' : 'offline']">
                  <i class="status-dot"></i> {{ headerState.statusText }}
                </span>
              </div>
            </header>

            <div class="message-list modern-scroll" ref="msgListRef">
              <div v-for="(msg, index) in messages" :key="index" :class="['msg-item', msg.role]">
                <template v-if="msg.role !== 'system-notice'">
                  <div class="msg-bubble" v-html="msg.text"></div>
                </template>
                <template v-else>
                  <div class="system-msg-content">{{ msg.text }}</div>
                </template>
              </div>

              <div class="msg-item system" v-if="isTyping">
                <div class="msg-bubble typing-dots">
                  <span></span><span></span><span></span>
                </div>
              </div>
            </div>

            <transition name="chat-fold" @after-enter="scrollToBottom">
              <footer class="chat-footer" v-if="showOptions && !isTyping">
                <div class="options-container">
                  <div v-for="(opt, idx) in currentNode.options" :key="idx" class="option-bubble"
                    @click="handleUserSelect(opt)">
                    {{ opt.label }}
                  </div>
                </div>
              </footer>
            </transition>
          </div>
        </section>

        <!-- [场景：照片墙 - 变更为“诚实模式”] -->
        <section v-else-if="viewMode === 'photos'" key="photos" class="scene-container">
          <div class="photo-gallery">
            <div class="gallery-header">
              <h2 class="gallery-title">素材匮乏</h2>
              <p class="gallery-subtitle">“翻遍了相册，发现咱俩以前可能真的不熟（误）”</p>
              <!-- 做个卡片,但没有图片部分，我要写小纸条 -->
              <div style="
    width: 300px;
    padding: 10px;
    background: rgba(255, 255, 255, 0.1);
    border: 1px solid rgba(255, 255, 255, 0.2);
    border-radius: 15px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
    color: #fff;
    font-family: '楷体';
    margin: 20px auto;
    text-align: left; /* 显式设置为靠左对齐 */
  ">
                <p style="font-size: 1.2rem;text-align: center; margin: 0 0 10px;">Notice</p>
                <p style="font-size: 0.9rem; color: rgba(255, 255, 255, 0.8);">
                  我对咱的关系没有那么死板，或者说，我没那么坚定，我刚开始同意，因为怕拒绝之后咱俩有鸿沟，不能像之前那样想聊什么聊什么，所以骗你<br>
                  既然我持保留态度，你持拒绝态度，那就想开点,都过去了<br>
                  <strong>所以以下内容，你用老同学的身份去看，我应该也没做什么过分的事情吧，你就当那些是跨越性别的友谊吧 生日快乐 </strong>
                </p>
              </div>
            </div>

            <div class="polaroid-wrapper">
              <div v-for="(photo, pIdx) in photoWallData" :key="pIdx" class="polaroid-card" :style="photo.customStyle">

                <div v-if="photo.type === 'placeholder'" class="empty-photo">
                  <span class="error-icon">⚠️</span>
                  <p class="error-text">{{ photo.errorText }}</p>
                </div>
                <img v-else :src="photo.url" />

                <p class="polaroid-desc">{{ photo.desc }}</p>
              </div>
            </div>

            <div class="apology-area">
              <button class="back-btn" @click="viewMode = 'chat'">
                THE END
              </button>
            </div>
          </div>
        </section>

        <!-- [场景：留言 1 - 温馨] -->
        <section v-else-if="viewMode === 'finale'" key="finale" class="scene-container">
          <div class="glass-card finale-card">
            <div class="badge">Happy Birthday</div>
            <h2>愿你岁岁平安</h2>
            <div class="divider"></div>
            <p class="gold-text">20年前你的诞生，后来会让很多人感到幸福和快乐<br>
              愿你眼底藏星，心中有海<br>
              在人间烟火里安稳，在岁月长途中自由<br>
              不被世俗磨去棱角，不被生活耗尽热忱<br>
              往后每一岁，都奔赴在自己的热爱里<br>
              生日快乐，愿你永远被偏爱<br>
              哪怕前路漫漫，也请记得有人曾为你送上这份笨拙的祝福</p>
            <button class="start-btn outline" @click="viewMode = 'chat'">重温记录</button>
          </div>
        </section>

        <!-- [场景：留言 2 - 遗憾] -->
        <section v-else-if="viewMode === 'last_finale'" key="last_finale" class="scene-container">
          <div class="glass-card finale-card deactivated-card">
            <div class="badge gray">AS YOU WISH</div>
            <h2>如您所愿 我们不会再次见面了</h2>
            <div class="divider"></div>
            <p class="gray-text">这份祝福会永远留在某个不知名的角落。<br>
              以后你会以自己的方式度过充实且有趣的一生<br>
              你可能会为某人改变，但与我无关了<br>
              只是前女友|精心制作调试个程序礼物算不算舔狗？<br>
              至少我觉得不算 礼物自身充满形式主义<br>
              更多是对过往经历的谢幕礼<br>
              如果您确实这么想，那对我来说又何尝不算解脱<br>
              山高路远，各自安好。</p>
            <button class="start-btn outline" @click="resetToHome">结束</button>
          </div>
        </section>
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, nextTick } from 'vue'

import 晚霞 from './img/IMG_20260113_091056_233.jpg'
import 头像 from './img/头像.png'
import 落日辰 from './img/20250916_082000.jpg'
import 车站 from './img/20250819_203438.jpg'
import 照片 from './img/blurred_IMG20220716201738.jpg'
import 聊天记录 from './img/record.jpg'
import 想法 from './img/20251116_080337.jpg'
import future from './img/dac8c0d6-d362-4bad-aba4-8dce2dd709d6.jpeg'

const headerState = reactive({
  nickname: '未知用户',
  statusText: '在线',
  isOnline: true
})

const setOfflineStyle = () => {
  headerState.nickname = '该账户已注销'; headerState.statusText = '离线'; headerState.isOnline = false
}
const setOnlineStyle = () => {
  headerState.nickname = '未知用户'; headerState.statusText = '在线'; headerState.isOnline = true
}

const plotTree = {
  'start': {
    replies: ["你好，我添加了你的好友，现在我们可以开始聊天啦。"],
    options: [
      { label: "[获取生日祝福]", userMsg: "我准备好拆礼物了", nextId: 'change2' },
      { label: "[看看你在搞什么花样]", userMsg: "你在吗？能看到消息吗", nextId: 'start1' }
    ]
  },
  'start1': {
    replies: ["[system_msg] 对方网络状态异常", "[system_msg]2021年12月?日 08:??", "我想问你个问题，你想我不想？"],
    options: [
      { label: "想", userMsg: "你说呢", nextId: 'daily1' },
      { label: "不想", userMsg: "可能以前想过，但是现在不想", nextId: 'start11' }
    ]
  },
  'start11': {
    replies: ["真不想啊？"],
    options: [
      { label: "骗你的", userMsg: "骗你的，肯定想", nextId: 'start11a' },
      { label: "不想", userMsg: "没错，你有什么好想的", nextId: 'start1111' },
    ]
  },
  'start11a': {
    replies: ["[system_msg]对方网络恢复", "[system_msg]2026年3月3日 08:??", "差点信了，那我可要伤心了"],
    options: [
      { label: "安慰", userMsg: ["开玩笑啦", "对了", "我过生日，你的祝福呢", "信不信许愿你倒霉一分钟"], nextId: 'start11ab' },
    ]
  },
  'start11ab': {
    replies: ["哇，狠毒，不过许愿还是想点对自己好的事吧hh", "生日快乐", "感觉我没什么审美，设计的照片墙不太满意，所以给你看祝福语", "[system_msg] 正在加载“留言”模块...", "[system_msg] 模块加载完成"],
    options: [{ label: "查看", userMsg: "", nextId: 'goto_finale' }]
  },
  'start1111': {
    replies: ["确定？"],
    options: [
      { label: "反悔", userMsg: "骗你的，肯定想", nextId: 'start11a' },
      { label: "确定，你好烦", userMsg: "我确定，我说几遍，别烦我行吗", nextId: 'start111' },
    ]
  },
  'start111': {
    replies: ["[system_msg]2026年2月15日 08:??", "hh", "好吧好吧，这才是我心里觉得你会说的话", "看来我也终于猜对了一次", "没事，生日祝福也不是熟人的特权吧", "祝福完我就跑，嘿嘿", "[system_msg] 正在加载“留言”模块...", "[system_msg] 模块加载完成", "[system_msg]系统提示：以上内容均为定时发送"],
    options: [{ label: "查看", userMsg: "", nextId: 'goto_last_finale' }]
  },
  'change2': {
    replies: ["[system_msg]2026年3月3日 08:??", "生日快乐", "由于找了半天也没看见什么合适的礼物，所以自己做一个网页小程序当作礼物吧，如果落差太大，实在抱歉"],
    options: [{ label: "安慰对方", userMsg: "没事，知道你就是这点不行，我很满意了", nextId: 'change4' }]
  },
  'change4': {
    replies: ["[到底是那点不行啊喂]", "算了，不跟你计较了", "我做了一张照片墙，虽然我不喜欢拍照 所以导致咱俩以前基本没拍过照片", "不过都是明信片，表达一下祝福", "还有个小彩蛋，是留言板，想看看不想看算了，内容不多"],
    options: [
      { label: "看照片墙", userMsg: "真的假的啊，你还会做照片墙呢，来吧来吧", nextId: 'change5' },
      { label: "看留言板", userMsg: "我还是想看照片墙", nextId: 'change5' },
      { label: "询问不喜欢拍照的原因", userMsg: "你为什么不喜欢拍照啊", nextId: 'change3' }
    ]
  },
  'change5': {
    replies: ["好的", "[system_msg] 正在载入“照片墙”组件...", "[system_msg] 载入成功"],
    options: [{ label: "查看", userMsg: "", nextId: 'goto_photos' }]
  },
  'change3': {
    replies: ["嗯......", "我觉得照片是离别的产物吧，觉得下一秒就要散了，所以不想去涉及", "虽然真的离别之后，没照片更难过", "以后说不定会忘得就只剩个名字", "不过还是一直这么想，改不过来，这就是人类的脑子，神奇吧"],
    options: [{ label: "[岔开话题]", userMsg: "让我看看你做的照片墙怎么样吧", nextId: 'change5' }]
  },
  'daily1': {
    replies: ["我咋知道"],
    options: [{ label: "直抒胸臆", userMsg: "想啊", nextId: 'daily2' }]
  },
  'daily2': {
    replies: ["问你个问题，我们班有男生追我，我该不该答应他"],
    options: [{ label: "询问对方的想法", userMsg: "啊？那你感觉他咋样？", nextId: 'daily3' }]
  },
  'daily3': {
    replies: ["你指哪方面"],
    options: [{ label: "你喜欢他吗", userMsg: "就是你喜不喜欢他", nextId: 'daily4' }]
  },
  'daily4': {
    replies: ["你想我喜欢还是不喜欢", "好吧好吧其实我不太喜欢", "<div style='color:gray; font-style: italic;'>[此条数据记录已丢失]</div>", "干嘛不理我？"],
    options: [{ label: "[脑子宕机ing...]", userMsg: ["不知道怎么回,感觉我也挺喜欢你的", "只不过当朋友更合适"], nextId: 'daily5' }]
  },
  'daily5': {
    replies: ["[system_msg] 08:43", "<div style='color:gray; font-style: italic;'>[此条数据记录已丢失]</div>", "[system_msg] 过往经历回溯已结束", "[system_msg]2026年2月14日 08:??",],
    options: [{ label: "？", userMsg: ["?", "提这事干嘛"], nextId: 'change1' }]
  },
  'change1': {
    replies: ["单纯想复刻一下以前的体验", "开个玩笑啦，别生气，只是再看到这段记录 感觉我当时都没get到你想表达的", "欸嘿~"],
    options: [{ label: "不想理你", userMsg: "无聊", nextId: 'change11' }]
  },
  'change11': {
    replies: ["嘿嘿", "[system_msg]2026年3月3日 08:??", "好久不见啊，生日快乐"],
    options: [{ label: "谢谢", userMsg: "谢谢", nextId: 'no_thanks' }]
  },
  'no_thanks': {
    replies: ["你这个谢谢让我有点生疏", "咱这个关系就不用说这个了吧"],
    options: [
      { label: "好", userMsg: "好的，以后不会说了", nextId: 'gift1' },
      { label: "不好", userMsg: "咱什么关系？", nextId: 'no_thanks_end' }
    ]
  },
  'no_thanks_end': {
    replies: ["为实现共产主义伟大目标而并肩奋斗的无产阶级革命战友", "以理想为旗、以奋斗为路的新时代青年伙伴。", "同守初心、共担使命的红色事业同志。"],
    options: [{ label: "彳亍", userMsg: ["...", "你好烦"], nextId: 'gift1' }]
  },
  'gift1': {
    replies: ["至于礼物，虽然之前说要问你地址，给你买个什么", "不过找了挺久也没感觉有合适的，想着自己亲手做的东西，当礼物送人会比较有意义", "所以最后就想着靠自己会的知识做个程序，给你留下了这些记录，算是一种比较另类的小册子吧", "也算是结合了我各方面的优缺点吧"],
    options: [
      { label: "[吐槽]：优点在哪", userMsg: ["没事，我比较傻，就吃这一套", "话说咱俩刚认识的时候什么样"], nextId: 'before' },
      { label: "安慰", userMsg: "没事，挺不错的，你随便买点东西给我才要骂你呢", nextId: 'after' }
    ]
  },
  'before': {
    replies: ["最开始有交集好像是英语老师把我分配给你背书过关", "当时感觉你还怪好说话，过关背书卡的不严", "之前天天见的时候也没问过你喜欢什么，所以现在只能靠“俺寻思之力了”"],
    options: [{ label: "继续话题", userMsg: ["那时候你多像个人", "毕业了什么都不愿意干", "说了也不听，听了也不干"], nextId: 'before1' }]
  },
  'before1': {
    replies: ["当时想法多单纯，你努力要去一高，我纯粹混日子", "而且在学校，学习总能当作聊天的借口", "卷子做了没？借我抄抄，给我看看，有吃的没，给我分点", "然后呢，然后就没有然后了", "所有都结束了，共患难的关系自然没理由继续存在了", "所以现在说什么呢，我也不知道，好像该说生日快乐吧", "[system_msg] 正在加载“留言”模块...", "[system_msg] 模块加载完成"],
    options: [{ label: "...", userMsg: "", nextId: 'goto_finale' }]
  },
  'after': {
    replies: ["真不生气啊，那就行"],
    options: [{ label: "抛出话题", userMsg: ["以前你多像个人", "现在什么都不愿意干", "说了也不听，听了也不干"], nextId: 'before1' }]
  },
}

const viewMode = ref('home')
const activeNodeId = ref('start')
const messages = reactive([])
const isTyping = ref(false)
const showOptions = ref(false)
const msgListRef = ref(null)

const currentNode = computed(() => plotTree[activeNodeId.value])

const startChat = async () => {
  setOnlineStyle(); viewMode.value = 'chat'; messages.length = 0; await sleep(600); runNode('start')
}

const resetToHome = () => {
  setOnlineStyle(); viewMode.value = 'home'
}

const runNode = async (nodeId) => {
  activeNodeId.value = nodeId; const node = plotTree[nodeId]; showOptions.value = false
  for (const text of node.replies) {
    if (!text || text.trim() === '') continue
    const isSystemMsg = text.startsWith('[system_msg]')
    if (isSystemMsg) {
      messages.push({ role: 'system-notice', text: text.replace('[system_msg]', '') })
    } else {
      isTyping.value = true; await sleep(Math.random() * 400 + 800); isTyping.value = false
      messages.push({ role: 'system', text })
    }
    scrollToBottom(); await sleep(400)
  }
  showOptions.value = true
}

const handleUserSelect = async (opt) => {
  showOptions.value = false
  const userMsgs = Array.isArray(opt.userMsg) ? opt.userMsg : [opt.userMsg]
  for (const text of userMsgs) {
    if (!text || text.trim() === '') continue
    messages.push({ role: 'user', text }); scrollToBottom(); await sleep(600)
  }
  await sleep(600)
  if (opt.nextId === 'start111') { setTimeout(() => { setOfflineStyle() }, 1200) }
  if (opt.nextId === 'goto_photos') { viewMode.value = 'photos'; return }
  if (opt.nextId === 'goto_finale') { viewMode.value = 'finale'; return }
  if (opt.nextId === 'goto_last_finale') { viewMode.value = 'last_finale'; return }
  runNode(opt.nextId)
}

const scrollToBottom = () => {
  nextTick(() => { if (msgListRef.value) msgListRef.value.scrollTo({ top: msgListRef.value.scrollHeight, behavior: 'smooth' }) })
}

const sleep = (ms) => new Promise(r => setTimeout(r, ms))

// 照片墙诚实版数据
const photoWallData = [
  { type: 'placeholder', errorText: '加载失败: 查无此合照', desc: "2021年, 咱俩光顾着背书了", customStyle: { top: '10%', left: '5%', transform: 'rotate(0deg)' } },
  { url: 晚霞, desc: "实在找不到图，拿晚霞凑个数", customStyle: { top: '26%', left: '16%', transform: 'rotate(5deg)' } },
  { type: 'placeholder', errorText: '404 NOT FOUND', desc: "这里本该有张你趴桌子上睡觉的英俊照片", customStyle: { top: '45%', left: '3%', transform: 'rotate(-2deg)' } },
  { url: 车站, desc: "这张不错，虽然没你", customStyle: { top: '51%', left: '30%', transform: 'rotate(0deg)' } },
  { type: 'placeholder', errorText: '文件已过期', desc: "当时吃饭想拍来着，感觉你腿很白(欠打了)，不过没找到机会", customStyle: { top: '10%', left: '70%', transform: 'rotate(2deg)' } },
  { type: 'placeholder', errorText: '文件不存在', desc: "??岁的合照", customStyle: { top: '55%', left: '70%', transform: 'rotate(-5deg)', border: '2px dashed #ffcc33' } },
  { url: 聊天记录, desc: "不知道什么时候截的图，也是网盘的自动备份相册里面看见的", customStyle: { top: '65%', left: '17%', transform: 'rotate(-3deg)' } },
  { url: 照片, desc: "留吧，咱俩有点前朝余孽（是这个词吗）的感觉；删吧，感觉你当时还挺在意这张的；不想让留的话，直接发消息跟我说", customStyle: { top: '60%', left: '85%', transform: 'rotate(3deg)' } },
  { url: 落日辰, desc: "网上看见的 因为名字 【落日辰】感觉挺熟（可能错别字）", customStyle: { top: '12%', left: '28%', transform: 'rotate(-2deg)' } },
  { url: 想法, desc: "咱俩牵过手吗，那次算吗", customStyle: { top: '15%', left: '85%', transform: 'rotate(2deg)' } },
  { url: future, desc: "都过去了", customStyle: { top: '55%', left: '52%', transform: 'rotate(1deg)' } },
]
</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
  width: 100vw;
  height: 100vh;
  overflow: hidden !important;
  background: #000;
}

* {
  box-sizing: border-box;
}
</style>

<style scoped>
.app-shell {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 1;
}

.background-system {
  position: absolute;
  inset: 0;
  z-index: 1;
  overflow: hidden;
}

.bg-layer {
  position: absolute;
  inset: 0;
  background-size: cover;
  background-position: center;
}

.bg-mask-static {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  z-index: 2;
}

.main-stage {
  position: relative;
  z-index: 10;
  width: 100%;
  height: 100%;
}

.scene-container {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

.gpu-accelerate {
  will-change: transform, opacity;
  transform: translateZ(0);
}

/* --- 聊天窗口 --- */
.chat-window {
  width: 100%;
  max-width: 500px;
  height: 85vh;
  display: flex;
  flex-direction: column;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 30px;
  overflow: hidden;
  color: #fff;
  box-shadow: 0 30px 60px rgba(0, 0, 0, 0.4);
}

.chat-header {
  flex-shrink: 0;
  padding: 15px 20px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  text-align: center;
  background: rgba(0, 0, 0, 0.2);
}

.nickname {
  font-size: 1.1rem;
  font-weight: 600;
  margin-bottom: 4px;
  transition: color 0.5s;
}

.deactivated-text {
  color: rgba(255, 255, 255, 0.4);
}

.status-row {
  height: 14px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.status-text {
  font-size: 0.75rem;
  display: flex;
  align-items: center;
  gap: 5px;
}

.status-text.online {
  color: rgba(255, 255, 255, 0.6);
}

.status-text.offline {
  color: rgba(255, 255, 255, 0.25);
}

.status-dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  display: inline-block;
  transition: all 0.5s;
}

.online .status-dot {
  background: #4caf50;
  box-shadow: 0 0 8px #4caf50;
}

.offline .status-dot {
  background: #666;
}

.message-list {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 12px;
  scrollbar-gutter: stable;
}

.modern-scroll::-webkit-scrollbar {
  width: 6px;
}

.modern-scroll::-webkit-scrollbar-thumb {
  background: rgba(255, 204, 51, 0.2);
  border-radius: 10px;
}

.msg-item {
  display: flex;
  width: 100%;
}

.msg-item.user {
  justify-content: flex-end;
}

.msg-item.system-notice {
  justify-content: center;
  margin: 2px 0;
}

.system-msg-content {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.35);
  text-align: center;
  max-width: 80%;
  line-height: 1.4;
}

.msg-bubble {
  max-width: 85%;
  padding: 10px 16px;
  border-radius: 18px;
  line-height: 1.5;
  font-size: 1rem;
}

.system .msg-bubble {
  background: rgba(255, 255, 255, 0.15);
  border-bottom-left-radius: 4px;
}

.user .msg-bubble {
  background: #ffcc33;
  color: #000;
  border-bottom-right-radius: 4px;
  font-weight: 500;
}

.typing-dots {
  display: flex;
  gap: 4px;
  padding: 12px 18px !important;
}

.typing-dots span {
  width: 6px;
  height: 6px;
  background: #ffcc33;
  border-radius: 50%;
  animation: bounce 1.4s infinite ease-in-out;
}

.typing-dots span:nth-child(2) {
  animation-delay: 0.2s;
}

.typing-dots span:nth-child(3) {
  animation-delay: 0.4s;
}

@keyframes bounce {

  0%,
  80%,
  100% {
    transform: translateY(0);
  }

  40% {
    transform: translateY(-6px);
  }
}

.chat-footer {
  flex-shrink: 0;
  padding: 20px;
  background: rgba(0, 0, 0, 0.3);
  border-top: 1px solid rgba(255, 255, 255, 0.05);
  overflow: hidden;
}

.chat-fold-enter-active {
  transition: all 0.4s cubic-bezier(0.2, 0.8, 0.2, 1);
}

.chat-fold-leave-active {
  transition: all 0.3s ease-out;
}

.chat-fold-enter-from,
.chat-fold-leave-to {
  max-height: 0;
  opacity: 0;
  padding-top: 0 !important;
  padding-bottom: 0 !important;
  transform: translateY(10px);
}

.chat-fold-enter-to,
.chat-fold-leave-from {
  max-height: 400px;
  opacity: 1;
  transform: translateY(0);
}

.option-bubble {
  background: rgba(255, 204, 51, 0.1);
  border: 1px solid #ffcc33;
  color: #ffcc33;
  padding: 12px;
  border-radius: 15px;
  cursor: pointer;
  transition: 0.3s;
  text-align: center;
  margin-bottom: 10px;
}

.option-bubble:hover {
  background: #ffcc33;
  color: #000;
}

/* --- 照片墙求生欲样式 (新) --- */
.photo-gallery {
  width: 100%;
  height: 100%;
  position: relative;
}

.gallery-header {
  position: absolute;
  top: 5vh;
  text-align: center;
  width: 100%;
  z-index: 5;
}

.gallery-title {
  color: #ffcc33;
  font-size: 2.2rem;
  margin: 0;
  font-family: "楷体";
}

.gallery-subtitle {
  color: rgba(255, 204, 51, 0.7);
  font-size: 0.9rem;
  margin-top: 5px;
}

.polaroid-card {
  position: absolute;
  background: #fff;
  padding: 8px 8px 25px;
  width: 170px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.5);
}

.polaroid-card img {
  width: 100%;
  height: 140px;
  object-fit: cover;
}

.polaroid-desc {
  color: #333;
  text-align: center;
  margin-top: 10px;
  font-family: "楷体";
  font-size: 0.85rem;
  padding: 0 5px;
}

.empty-photo {
  width: 100%;
  height: 140px;
  background: #f0f0f0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border: 1px inset #ddd;
}

.error-icon {
  font-size: 2rem;
  margin-bottom: 10px;
}

.error-text {
  font-family: "Courier New", Courier, monospace !important;
  font-weight: bold;
  color: #888 !important;
  font-size: 0.7rem !important;
}

.apology-area {
  position: absolute;
  bottom: 5vh;
  width: 100%;
  display: flex;
  justify-content: center;
}

.back-btn {
  background: #ffcc33;
  color: #000;
  border: none;
  padding: 10px 30px;
  border-radius: 30px;
  cursor: pointer;
  font-weight: bold;
}

/* --- 其他 --- */
.scene-change-enter-active,
.scene-change-leave-active {
  transition: all 0.5s ease;
}

.scene-change-enter-from {
  opacity: 0;
  transform: scale(0.98);
}

.scene-change-leave-to {
  opacity: 0;
  transform: scale(1.02);
}

.glass-card {
  padding: 40px;
  text-align: center;
  max-width: 500px;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 30px;
  color: #fff;
}

.avatar-ripple {
  width: 110px;
  height: 110px;
  margin: 0 auto 25px;
  border-radius: 50%;
  padding: 5px;
  border: 2px solid #ffcc33;
}

.main-avatar {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  object-fit: cover;
}

.welcome-text {
  font-size: 2rem;
  margin-bottom: 10px;
}

.finale-card {
  position: relative;
  overflow: hidden;
}

.badge {
  position: absolute;
  top: 20px;
  right: -30px;
  transform: rotate(45deg);
  background: #ffcc33;
  color: #000;
  padding: 5px 40px;
  font-size: 0.7rem;
  font-weight: bold;
}

.badge.gray {
  background: #666;
  color: #eee;
}

.divider {
  height: 1px;
  background: linear-gradient(90deg, transparent, #ffcc33, transparent);
  margin: 25px 0;
}

.gold-text {
  color: #ffcc33;
  font-size: 1.1rem;
  line-height: 1.8;
}

.gray-text {
  color: #aaa;
  font-size: 1.1rem;
  line-height: 1.8;
}

.deactivated-card {
  border-color: rgba(255, 255, 255, 0.05);
  background: rgba(20, 20, 20, 0.4);
}

.start-btn {
  background: #ffcc33;
  color: #000;
  border: none;
  padding: 15px 50px;
  border-radius: 50px;
  font-weight: bold;
  cursor: pointer;
  transition: 0.3s;
  margin-top: 20px;
}

.start-btn.outline {
  background: transparent;
  border: 1px solid #ffcc33;
  color: #ffcc33;
}

.start-btn.outline:hover {
  background: #ffcc33;
  color: #000;
}
</style>