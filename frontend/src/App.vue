<script setup lang="ts">
import { ref, onMounted, watch, nextTick } from 'vue';
import { marked } from 'marked';

// 定义消息类型接口
interface Message {
  role: 'user' | 'assistant';
  content: string;
  timestamp?: number;
}

// 状态管理
const input = ref('');
const messages = ref<Message[]>([]);
const loading = ref(false);
const messagesEndRef = ref<HTMLDivElement | null>(null);

// 触摸事件状态变量
let isDragging = false;
let dragStartY = 0;
let containerScrollTop = 0;

// 触摸事件处理函数
const handleTouchStart = (e: TouchEvent) => {
  isDragging = true;
  if (e.touches && e.touches[0]) {
    dragStartY = e.touches[0].clientY;
  }
  const container = e.currentTarget as HTMLElement;
  containerScrollTop = container.scrollTop;
};

const handleTouchMove = (e: TouchEvent) => {
  if (!isDragging) return;
  const container = e.currentTarget as HTMLElement;
  if (e.touches && e.touches[0]) {
    const deltaY = e.touches[0].clientY - dragStartY;
    container.scrollTop = containerScrollTop - deltaY;
  }
};

const handleTouchEnd = () => {
  isDragging = false;
};

// 自动滚动到底部
const scrollToBottom = (smooth = true) => {
  nextTick(() => {
    messagesEndRef.value?.scrollIntoView({ behavior: smooth ? 'smooth' : 'auto' });
  });
};

// 监听消息变化，自动滚动
watch(
  () => messages.value.length,
  () => {
    scrollToBottom();
  }
);

// 发送消息函数
const sendMessage = async () => {
  if (!input.value.trim() || loading.value) return;

  // 获取或生成会话ID
  let sessionId = localStorage.getItem('chatSessionId');
  if (!sessionId) {
    sessionId = 'session_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
    localStorage.setItem('chatSessionId', sessionId);
  }

  // 添加用户消息
  const userMessage: Message = {
    role: 'user', 
    content: input.value,
    timestamp: Date.now()
  };
  messages.value = [...messages.value, userMessage];
  input.value = '';
  loading.value = true;

  try {
    const response = await fetch('/api/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ message: userMessage.content, session_id: sessionId }),
    });

    if (!response.ok) throw new Error('请求失败');

    const data = await response.json();
    const botMessage: Message = {
      role: 'assistant', 
      content: data.response,
      timestamp: Date.now()
    };
    messages.value = [...messages.value, botMessage];
  } catch (error) {
    const errorMsg: Message = {
      role: 'assistant',
      content: '❌ 请求出错，请稍后重试。',
      timestamp: Date.now()
    };
    messages.value = [...messages.value, errorMsg];
  } finally {
    loading.value = false;
  }
};

// 处理按键事件
const handleKeyPress = (e: KeyboardEvent) => {
  if (e.key === 'Enter' && !e.shiftKey) {
    e.preventDefault();
    sendMessage();
  }
};

// 自定义Markdown渲染配置
marked.use({
  breaks: true,
  gfm: true,
});

// 渲染Markdown内容
const renderMarkdown = (content: string) => {
  return marked(content);
};

// 清除会话历史
const clearSession = () => {
  messages.value = [];
  localStorage.removeItem('chatSessionId');
  // 重新加载欢迎消息
  messages.value = [
    {
      role: 'assistant',
      content: '👋 您好！我是智能数据分析助手。请问您需要了解数据库中的哪些信息？',
      timestamp: Date.now()
    }
  ];
  scrollToBottom(false);
};

// 组件挂载时初始化
onMounted(() => {
  // 加载欢迎消息
  messages.value = [
    {
      role: 'assistant',
      content: '👋 您好！我是智能数据分析助手。请问您需要了解数据库中的哪些信息？',
      timestamp: Date.now()
    }
  ];
  scrollToBottom(false);
});
</script>

<template>
  <div class="chat-container">
    <!-- 头部区域 -->
    <header class="chat-header">
      <div class="header-content">
        <div class="header-logo">
          <svg viewBox="0 0 24 24" width="24" height="24" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round" stroke-linejoin="round">
            <polygon points="12 2 2 7 12 12 22 7 12 2"></polygon>
            <polyline points="2 17 12 22 22 17"></polyline>
            <polyline points="2 12 12 17 22 12"></polyline>
          </svg>
        </div>
        <h1 class="header-title">智能数据分析助手</h1>
        <button class="clear-session-btn" @click="clearSession">
          <svg viewBox="0 0 24 24" width="20" height="20" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round" stroke-linejoin="round">
            <line x1="18" y1="6" x2="6" y2="18"></line>
            <line x1="6" y1="6" x2="18" y2="18"></line>
          </svg>
        </button>
      </div>
    </header>

    <!-- 对话区域 -->
    <main 
      class="chat-messages-container"
      @touchstart="handleTouchStart"
      @touchmove="handleTouchMove"
      @touchend="handleTouchEnd"
    >
      <!-- 欢迎提示 -->
      <div class="welcome-banner">
        <div class="welcome-icon">💡</div>
        <div class="welcome-text">
          您可以询问关于数据库结构、数据分析或查询优化的问题
        </div>
      </div>

      <!-- 消息列表 -->
      <div class="messages-list">
        <div 
          v-for="(msg, i) in messages" 
          :key="i"
          :class="['message-wrapper', msg.role === 'user' ? 'user-message' : 'assistant-message']"
        >
          <!-- 角色头像 -->
          <div :class="['message-avatar', msg.role]">
            {{ msg.role === 'user' ? '👤' : '🤖' }}
          </div>
          
          <!-- 消息内容 -->
          <div 
            :class="['message-content', msg.role]
          "
          >
            <div v-html="renderMarkdown(msg.content)"></div>
            
            <!-- 时间戳 -->
            <div class="message-time">
              {{ new Date(msg.timestamp || Date.now()).toLocaleTimeString('zh-CN', { 
                hour: '2-digit', 
                minute: '2-digit'
              }) }}
            </div>
          </div>
        </div>
      </div>

      <!-- 加载状态 -->
      <div v-if="loading" class="loading-indicator">
        <div class="loading-dots">
          <div class="dot"></div>
          <div class="dot"></div>
          <div class="dot"></div>
        </div>
        <span class="loading-text">正在分析...</span>
      </div>

      <!-- 滚动锚点 -->
      <div ref="messagesEndRef" class="scroll-anchor" />
    </main>

    <!-- 输入区域 -->
    <footer class="chat-input-container">
      <div class="input-wrapper">
        <textarea
          v-model="input"
          @keydown="handleKeyPress"
          placeholder="请输入您的问题..."
          :disabled="loading"
          class="chat-input"
          rows="1"
        />
        <button
          @click="sendMessage"
          :disabled="loading || !input.trim()"
          class="send-button"
          :class="{ disabled: loading || !input.trim() }"
        >
          <svg v-if="!loading" viewBox="0 0 24 24" width="20" height="20" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round" stroke-linejoin="round">
            <line x1="22" y1="2" x2="11" y2="13"></line>
            <polygon points="22 2 15 22 11 13 2 9 22 2"></polygon>
          </svg>
          <svg v-else class="loading-spinner" viewBox="0 0 24 24" width="20" height="20">
            <circle cx="12" cy="12" r="10" fill="none" stroke="currentColor" stroke-width="3" stroke-dasharray="50,50" stroke-linecap="round" transform="rotate(0 12 12)"></circle>
          </svg>
        </button>
      </div>
      <div class="input-hint">
        按 Enter 发送，Shift + Enter 换行
      </div>
    </footer>
  </div>
</template>

<style scoped>
/* 主题色彩变量 */
:root {
  --primary-color: #4361ee;
  --primary-light: #4895ef;
  --primary-dark: #3a0ca3;
  --secondary-color: #4cc9f0;
  --success-color: #4cc9f0;
  --danger-color: #f72585;
  --warning-color: #ffd60a;
  --text-primary: #2d3748;
  --text-secondary: #4a5568;
  --text-muted: #718096;
  --bg-primary: #ffffff;
  --bg-secondary: #f7fafc;
  --bg-tertiary: #edf2f7;
  --border-color: #e2e8f0;
  --shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
}

/* 主容器 */
.chat-container {
  height: 100vh;
  display: flex;
  flex-direction: column;
  background-color: var(--bg-secondary);
  color: var(--text-primary);
}

/* 头部区域 */
.chat-header {
  background-color: var(--bg-primary);
  border-bottom: 1px solid var(--border-color);
  box-shadow: var(--shadow-sm);
  padding: 1rem 0;
}

.header-content {
  max-width: 800px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 1.5rem;
}

.header-logo {
  color: var(--primary-color);
  width: 32px;
  height: 32px;
}

.header-title {
  font-size: 1.5rem;
  font-weight: 600;
  color: var(--text-primary);
  margin: 0;
}

.clear-session-btn {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  border: none;
  background-color: var(--bg-tertiary);
  color: var(--text-secondary);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
}

.clear-session-btn:hover {
  background-color: var(--border-color);
  color: var(--text-primary);
  transform: rotate(15deg);
}

.clear-session-btn:active {
  transform: rotate(15deg) scale(0.95);
}

/* 消息容器 */
.chat-messages-container {
  flex: 1;
  overflow-y: auto;
  max-width: 800px;
  width: 100%;
  margin: 0 auto;
  padding: 1.5rem;
  scroll-behavior: smooth;
  -webkit-overflow-scrolling: touch;
}

.chat-messages-container::-webkit-scrollbar {
  width: 6px;
}

.chat-messages-container::-webkit-scrollbar-track {
  background: var(--bg-secondary);
}

.chat-messages-container::-webkit-scrollbar-thumb {
  background: var(--border-color);
  border-radius: 3px;
}

.chat-messages-container::-webkit-scrollbar-thumb:hover {
  background: var(--text-muted);
}

/* 欢迎提示 */
.welcome-banner {
  display: flex;
  align-items: center;
  background-color: var(--bg-tertiary);
  border-radius: 12px;
  padding: 1rem;
  margin-bottom: 1.5rem;
  border: 1px solid var(--border-color);
}

.welcome-icon {
  font-size: 1.5rem;
  margin-right: 0.75rem;
}

.welcome-text {
  color: var(--text-secondary);
  font-size: 0.875rem;
  flex: 1;
}

/* 消息列表 */
.messages-list {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

/* 消息包装器 */
.message-wrapper {
  display: flex;
  align-items: flex-start;
  gap: 0.75rem;
  max-width: 80%;
}

.user-message {
  margin-left: auto;
  flex-direction: row-reverse;
}

.assistant-message {
  margin-right: auto;
}

/* 消息头像 */
.message-avatar {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background-color: var(--bg-tertiary);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  font-size: 1.25rem;
  box-shadow: var(--shadow-sm);
}

.message-avatar.user {
  background-color: var(--primary-color);
  color: white;
}

.message-avatar.assistant {
  background-color: var(--secondary-color);
  color: white;
}

/* 消息内容 */
.message-content {
  position: relative;
  padding: 0.75rem 1rem;
  border-radius: 18px;
  font-size: 0.9375rem;
  line-height: 1.5;
  word-wrap: break-word;
  box-shadow: var(--shadow-sm);
  max-width: calc(100% - 48px);
}

.message-content.user {
  background-color: var(--primary-color);
  color: white;
  border-bottom-right-radius: 6px;
}

.message-content.assistant {
  background-color: var(--bg-primary);
  color: var(--text-primary);
  border: 1px solid var(--border-color);
  border-bottom-left-radius: 6px;
}

/* 消息时间戳 */
.message-time {
  font-size: 0.75rem;
  opacity: 0.7;
  margin-top: 0.25rem;
  text-align: right;
}

/* 加载指示器 */
.loading-indicator {
  display: flex;
  align-items: center;
  margin: 1rem 0;
  color: var(--text-muted);
  font-size: 0.875rem;
}

.loading-dots {
  display: flex;
  gap: 0.25rem;
  margin-right: 0.5rem;
}

.dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background-color: var(--text-muted);
  animation: loading 1.4s infinite ease-in-out both;
}

.dot:nth-child(1) {
  animation-delay: -0.32s;
}

.dot:nth-child(2) {
  animation-delay: -0.16s;
}

@keyframes loading {
  0%, 80%, 100% {
    transform: scale(0);
  }
  40% {
    transform: scale(1);
  }
}

/* 加载旋转图标 */
.loading-spinner {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

/* 输入区域 */
.chat-input-container {
  background-color: var(--bg-primary);
  border-top: 1px solid var(--border-color);
  padding: 1rem 0;
  box-shadow: 0 -1px 3px rgba(0, 0, 0, 0.05);
}

.input-wrapper {
  max-width: 800px;
  margin: 0 auto;
  padding: 0 1.5rem;
  display: flex;
  gap: 0.75rem;
  align-items: flex-end;
}

.chat-input {
  flex: 1;
  min-height: 48px;
  max-height: 120px;
  padding: 0.75rem 1rem;
  border: 2px solid var(--border-color);
  border-radius: 24px;
  font-size: 0.9375rem;
  line-height: 1.5;
  resize: none;
  outline: none;
  background-color: var(--bg-primary);
  color: var(--text-primary);
  transition: all 0.2s ease;
  font-family: inherit;
}

.chat-input:focus {
  border-color: var(--primary-color);
  box-shadow: 0 0 0 3px rgba(67, 97, 238, 0.1);
}

.chat-input:disabled {
  background-color: var(--bg-tertiary);
  cursor: not-allowed;
}

.chat-input::placeholder {
  color: var(--text-muted);
}

/* 发送按钮 */
.send-button {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  border: none;
  background-color: var(--primary-color);
  color: white;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  box-shadow: var(--shadow-sm);
  flex-shrink: 0;
}

.send-button:hover:not(.disabled) {
  background-color: var(--primary-dark);
  transform: translateY(-1px);
  box-shadow: var(--shadow-md);
}

.send-button:active:not(.disabled) {
  transform: translateY(0);
  box-shadow: var(--shadow-sm);
}

.send-button.disabled {
  background-color: var(--bg-tertiary);
  color: var(--text-muted);
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}

/* 输入提示 */
.input-hint {
  max-width: 800px;
  margin: 0.5rem auto 0;
  padding: 0 1.5rem;
  font-size: 0.75rem;
  color: var(--text-muted);
  text-align: center;
}

/* Markdown 样式 */
.message-content :deep(p) {
  margin: 0 0 0.5rem 0;
}

.message-content :deep(p:last-child) {
  margin-bottom: 0;
}

.message-content :deep(pre) {
  background-color: rgba(0, 0, 0, 0.05);
  padding: 0.75rem;
  border-radius: 8px;
  overflow-x: auto;
  margin: 0.5rem 0;
  font-size: 0.875rem;
  line-height: 1.4;
  font-family: 'SF Mono', Monaco, 'Cascadia Code', 'Roboto Mono', Consolas, 'Courier New', monospace;
}

.message-content.user :deep(pre) {
  background-color: rgba(255, 255, 255, 0.1);
}

.message-content :deep(code) {
  font-family: 'SF Mono', Monaco, 'Cascadia Code', 'Roboto Mono', Consolas, 'Courier New', monospace;
  font-size: 0.875em;
  padding: 0.125rem 0.25rem;
  border-radius: 4px;
  background-color: rgba(0, 0, 0, 0.05);
}

.message-content.user :deep(code) {
  background-color: rgba(255, 255, 255, 0.1);
}

.message-content :deep(pre code) {
  background-color: transparent;
  padding: 0;
}

.message-content :deep(img) {
  max-width: 100%;
  border-radius: 8px;
  margin: 0.5rem 0;
  border: 1px solid var(--border-color);
}

.message-content.user :deep(img) {
  border-color: rgba(255, 255, 255, 0.2);
}

.message-content :deep(blockquote) {
  border-left: 4px solid var(--primary-light);
  padding-left: 0.75rem;
  margin: 0.5rem 0;
  color: inherit;
  opacity: 0.8;
}

.message-content :deep(ul),
.message-content :deep(ol) {
  margin: 0.5rem 0;
  padding-left: 1.5rem;
}

.message-content :deep(li) {
  margin-bottom: 0.25rem;
}

.message-content :deep(h1),
.message-content :deep(h2),
.message-content :deep(h3) {
  margin: 0.75rem 0 0.5rem 0;
  font-weight: 600;
}

.message-content :deep(h1) {
  font-size: 1.5rem;
}

.message-content :deep(h2) {
  font-size: 1.25rem;
}

.message-content :deep(h3) {
  font-size: 1.125rem;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .header-content,
  .chat-messages-container,
  .input-wrapper,
  .input-hint {
    padding-left: 1rem;
    padding-right: 1rem;
  }
  
  .header-title {
    font-size: 1.25rem;
  }
  
  .message-wrapper {
    max-width: 90%;
  }
  
  .chat-input {
    min-height: 44px;
  }
  
  .send-button {
    width: 44px;
    height: 44px;
  }
}

@media (max-width: 480px) {
  .chat-messages-container {
    padding-top: 1rem;
    padding-bottom: 1rem;
  }
  
  .message-content {
    font-size: 0.875rem;
    padding: 0.625rem 0.875rem;
  }
  
  .chat-input-container {
    padding: 0.75rem 0;
  }
}
</style>
