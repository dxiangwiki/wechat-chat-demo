<template>
  <div class="app-full">
    <!-- 顶部控制开关（固定在右上角，不遮挡主界面） -->
    <div class="top-switch">
      <label>
        <input v-model="showManagePanel" type="checkbox">
        显示消息管理面板
      </label>
      <label>
        <input v-model="useGraphMsgFirst" type="checkbox">
        优先使用图形化配置消息
      </label>
    </div>

    <!-- 微信主界面（全屏铺满，结构稳定） -->
    <div class="chat-wrap">
      <!-- 左侧侧边栏（固定结构，不挤压） -->
      <div class="left-side">
        <!-- 深色图标区（固定宽度） -->
        <div class="icon-bar">
          <div class="chat-icon"></div>
        </div>
        <!-- 聊天列表区（剩余宽度） -->
        <div class="list-body">
          <div class="search-box">
            <input placeholder="搜索" />
            <span class="add-btn">+</span>
          </div>
          <div class="chat-list">
            <div class="friend-item active">
              <img class="friend-avatar" src="https://picsum.photos/60?random=1" alt="">
              <div class="friend-info">
                <div class="friend-name">好友</div>
                <div class="friend-desc">在线聊天</div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 右侧聊天主界面（占满剩余空间） -->
      <div class="right-main">
        <div class="chat-header">
          <span>好友</span>
          <span class="more-btn">⋯</span>
        </div>
        <div class="chat-body" ref="chatBody">
          <template v-for="item in chatShowList" :key="item.id">
            <div class="time-divider">{{ item.time }}</div>
            <!-- 对方消息 -->
            <div class="msg-row left-msg" v-if="!item.isSelf">
              <img class="user-avatar" src="https://picsum.photos/60?random=1" alt="">
              <div class="msg-bubble left-bubble">{{ item.content }}</div>
            </div>
            <!-- 自己消息 -->
            <div class="msg-row right-msg" v-else>
              <div class="msg-bubble right-bubble">{{ item.content }}</div>
              <img class="user-avatar" src="https://picsum.photos/60?random=2" alt="">
            </div>
          </template>
        </div>
        <!-- 底部输入区（带图标） -->
        <div class="input-area">
          <div class="tool-bar">
            <span>😊</span>
            <span>📎</span>
            <span>🖼️</span>
          </div>
          <textarea v-model="sendText" @keyup.enter.prevent="sendMsg"></textarea>
          <div class="send-btn-wrap">
            <button @click="sendMsg">发送(S)</button>
          </div>
        </div>
      </div>
    </div>

    <!-- 消息管理面板（右侧悬浮） -->
    <div class="panel" v-if="showManagePanel">
      <h4>消息可视化配置面板</h4>
      <div class="form-row">
        <span>发送身份：</span>
        <label><input v-model="form.isSelf" type="radio" :value="false">对方</label>
        <label><input v-model="form.isSelf" type="radio" :value="true">自己</label>
      </div>
      <div class="form-row">
        <span>消息内容：</span>
        <input v-model="form.content" placeholder="输入消息内容" />
      </div>
      <div class="form-row">
        <span>发送时间：</span>
        <input v-model="form.time" placeholder="例：15:30" />
      </div>
      <button class="add-btn" @click="addMsg">添加消息</button>

      <div class="msg-list">
        <div class="msg-item" v-for="(m,i) in graphMsgList" :key="m.id">
          <span>{{ m.isSelf ? '自己' : '好友' }} {{ m.time }}</span>
          <span class="msg-content">{{ m.content }}</span>
          <button class="del-btn" @click="delMsg(i)">删除</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, nextTick, onMounted } from 'vue'
import { defaultMsgList } from './messageConfig.js'

// 控制开关
const showManagePanel = ref(true)
const useGraphMsgFirst = ref(false)

// 图形化消息列表
const graphMsgList = ref([])
const form = ref({ id: '', isSelf: false, content: '', time: '' })

// 最终渲染消息
const chatShowList = computed(() => useGraphMsgFirst.value ? graphMsgList.value : defaultMsgList)

// 发送消息
const sendText = ref('')
const chatBody = ref(null)
const sendMsg = () => {
  const val = sendText.value.trim()
  if (!val) return
  const now = new Date()
  const h = String(now.getHours()).padStart(2, '0')
  const m = String(now.getMinutes()).padStart(2, '0')
  graphMsgList.value.push({
    id: Date.now(),
    isSelf: true,
    content: val,
    time: `${h}:${m}`
  })
  sendText.value = ''
  nextTick(() => chatBody.value.scrollTop = chatBody.value.scrollHeight)
}

// 图形面板操作
const addMsg = () => {
  if (!form.value.content || !form.value.time) return
  graphMsgList.value.push({ ...form.value, id: Date.now() })
  form.value = { id: '', isSelf: false, content: '', time: '' }
}
const delMsg = (index) => graphMsgList.value.splice(index, 1)

onMounted(() => chatBody.value.scrollTop = chatBody.value.scrollHeight)
</script>

<style scoped>
/* 全局重置 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

/* 全屏容器：宽100%，高100vh，无溢出 */
.app-full {
  width: 100%;
  height: 100vh;
  overflow: hidden;
  background: #f0f0f0;
  position: relative;
}

/* 顶部开关（右上角，不遮挡主界面） */
.top-switch {
  position: fixed;
  top: 10px;
  right: 10px;
  z-index: 999;
  background: #fff;
  padding: 6px 12px;
  border-radius: 4px;
  display: flex;
  gap: 14px;
  font-size: 13px;
}

/* 主容器：flex 不挤压 */
.chat-wrap {
  width: 100%;
  height: 100%;
  display: flex;
}

/* 左侧栏（固定宽度，不压缩） */
.left-side {
  width: 300px;
  height: 100%;
  display: flex;
  background: #ededed;
  flex-shrink: 0;
}

.icon-bar {
  width: 55px;
  height: 100%;
  background: #2c3137;
  display: flex;
  justify-content: start;
  padding-top: 15px;
}

.chat-icon {
  width: 28px;
  height: 28px;
  background: #fff;
  border-radius: 2px;
}

.list-body {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.search-box {
  height: 50px;
  display: flex;
  align-items: center;
  padding: 0 10px;
  gap: 6px;
}

.search-box input {
  flex: 1;
  height: 30px;
  border: none;
  border-radius: 4px;
  padding: 0 8px;
  outline: none;
}

.add-btn {
  width: 30px;
  height: 30px;
  background: #ddd;
  border-radius: 4px;
  display: grid;
  place-content: center;
  cursor: pointer;
}

.chat-list {
  flex: 1;
  overflow-y: auto;
}

.friend-item {
  display: flex;
  align-items: center;
  padding: 10px;
  cursor: pointer;
}

.friend-item.active, .friend-item:hover {
  background: #d7d7d7;
}

.friend-avatar {
  width: 40px;
  height: 40px;
  border-radius: 4px;
  margin-right: 10px;
}

.friend-name {
  font-size: 14px;
}

.friend-desc {
  font-size: 12px;
  color: #888;
}

/* 右侧聊天区（占满剩余空间） */
.right-main {
  flex: 1;
  height: 100%;
  display: flex;
  flex-direction: column;
  background: #fafafa;
  overflow: hidden;
}

.chat-header {
  height: 50px;
  line-height: 50px;
  padding: 0 20px;
  background: #fff;
  border-bottom: 1px solid #e5e5e5;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 15px;
}

.chat-body {
  flex: 1;
  padding: 15px 30px;
  overflow-y: auto;
}

.time-divider {
  text-align: center;
  font-size: 12px;
  color: #999;
  margin: 10px 0;
}

.msg-row {
  display: flex;
  margin-bottom: 16px;
  align-items: flex-start;
}

.left-msg {
  justify-content: flex-start;
}

.right-msg {
  justify-content: flex-end;
}

.user-avatar {
  width: 36px;
  height: 36px;
  border-radius: 4px;
  flex-shrink: 0;
}

.msg-bubble {
  max-width: 60%;
  padding: 8px 12px;
  border-radius: 4px;
  font-size: 14px;
}

.left-bubble {
  background: #fff;
  border: 1px solid #e8e8e8;
  margin-left: 10px;
}

.right-bubble {
  background: #95ec69;
  margin-right: 10px;
}

/* 底部输入区（带图标） */
.input-area {
  height: 160px;
  background: #fff;
  border-top: 1px solid #e5e5e5;
  padding: 15px 20px;
}

.tool-bar {
  margin-bottom: 8px;
}

.tool-bar span {
  font-size: 18px;
  margin-right: 14px;
  cursor: pointer;
  color: #555;
}

.input-area textarea {
  width: 100%;
  height: 80px;
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 8px;
  outline: none;
  resize: none;
  box-sizing: border-box;
}

.send-btn-wrap {
  text-align: right;
  margin-top: 8px;
}

.send-btn-wrap button {
  padding: 5px 18px;
  background: #07c160;
  color: #fff;
  border: none;
  border-radius: 3px;
  cursor: pointer;
}

/* 消息管理面板 */
.panel {
  position: fixed;
  right: 15px;
  top: 60px;
  width: 300px;
  background: #fff;
  border-radius: 6px;
  padding: 15px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.15);
  max-height: 80vh;
  overflow-y: auto;
  z-index: 998;
}

.panel h4 {
  margin: 0 0 10px;
  text-align: center;
  font-size: 15px;
}

.form-row {
  margin: 8px 0;
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 13px;
}

.form-row input {
  padding: 3px 6px;
  border: 1px solid #ddd;
  border-radius: 3px;
}

.add-btn {
  background: #07c160;
  color: #fff;
  border: none;
  padding: 4px 10px;
  border-radius: 3px;
  margin: 6px 0 10px;
  cursor: pointer;
}

.msg-list {
  border-top: 1px solid #eee;
  padding-top: 10px;
}

.msg-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 6px 0;
  border-bottom: 1px solid #f5f5f5;
  font-size: 13px;
}

.msg-content {
  flex: 1;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  margin: 0 6px;
}

.del-btn {
  background: #f53f3f;
  color: #fff;
  border: none;
  padding: 2px 6px;
  border-radius: 3px;
  cursor: pointer;
}
</style>