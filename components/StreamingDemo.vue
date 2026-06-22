<template>
  <div class="wrapper">
    <div class="controls">
      <button class="btn btn-primary" :disabled="state === 'done'" @click="handleStream">
        {{ state === 'idle' ? 'ストリーミング開始' : state === 'paused' ? '再開' : '一時停止' }}
      </button>
      <a class="btn btn-ghost" :href="SOURCE_URL" target="_blank">ソースファイル ↗</a>
    </div>

    <div class="text-wrapper" ref="textWrapper">
      <span class="text-content">{{ text }}</span>
      <span v-if="state === 'streaming'" class="cursor">▋</span>
    </div>

    <div class="footer">
      <span class="status" :class="`status-${state}`">{{ statusLabel }}</span>
    </div>
  </div>
</template>

<script setup>
import {ref, computed, nextTick} from 'vue'

const FILE_URL = 'https://raw.githubusercontent.com/sun-choma/febi-generators-26.06.24/refs/heads/master/public/really-big-novel.txt'
const SOURCE_URL = 'https://github.com/sun-choma/febi-generators-26.06.24/blob/master/public/really-big-novel.txt'

// 'idle' | 'streaming' | 'paused' | 'done'
const state = ref('idle')
const text = ref('')
const textWrapper = ref(null)

let gen = null

const statusLabel = computed(() => ({
  idle: 'アイドル',
  streaming: 'ストリーミング中...',
  paused: '一時停止中',
  done: '完了',
}[state.value]))

async function* streamChunks(url) {
  const res = await fetch(url)
  const reader = res.body.getReader()
  const decoder = new TextDecoder()
  while (true) {
    const {done, value} = await reader.read()
    if (done) break
    const text = decoder.decode(value, {stream: true})
    for (let i = 0; i < text.length; i += 20) {
      yield text.slice(i, i + 20)
      await new Promise(r => setTimeout(r, 100))
    }
  }
}

async function runLoop() {
  for await (const chunk of gen) {
    while (state.value === 'paused') {
      await new Promise(r => setTimeout(r, 50))
    }
    text.value += chunk
    await nextTick()
    scrollToBottom()
  }
  state.value = 'done'
}

function scrollToBottom() {
  if (textWrapper.value) {
    textWrapper.value.scrollTop = textWrapper.value.scrollHeight
  }
}

function handleStream() {
  if (state.value === 'idle') {
    text.value = ''
    state.value = 'streaming'
    gen = streamChunks(FILE_URL)
    runLoop()
  } else if (state.value === 'streaming') {
    state.value = 'paused'
  } else if (state.value === 'paused') {
    state.value = 'streaming'
    nextTick(scrollToBottom)
  }
}
</script>

<style scoped>
.wrapper {
  font-family: ui-monospace, monospace;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.controls {
  display: flex;
  gap: 8px;
  align-items: center;
}

.btn {
  font-family: ui-monospace, monospace;
  font-size: 0.78rem;
  padding: 6px 16px;
  border-radius: 4px;
  cursor: pointer;
  border: 1.5px solid;
  transition: all 0.2s;
  background: transparent;
  text-decoration: none;
  white-space: nowrap;
}

.btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.btn-primary {
  border-color: #50fa7b;
  color: #50fa7b;
}

.btn-primary:hover:not(:disabled) {
  background: #0e3130;
}

.btn-secondary {
  border-color: #f1b529;
  color: #f1b529;
}

.btn-secondary:hover:not(:disabled) {
  background: #3d2e0e;
}

.btn-ghost {
  border-color: #6272a4;
  color: #6272a4;
  margin-left: auto;
}

.btn-ghost:hover {
  border-color: #f8f8f2;
  color: #f8f8f2;
}

.text-wrapper {
  border: 1.5px solid #6272a4;
  border-radius: 8px;
  height: 220px;
  overflow-y: auto;
  background: #21222c;
  padding: 12px;
  font-size: 0.75rem;
  line-height: 1.6;
  color: #f8f8f2;
  white-space: pre-wrap;
  word-break: break-word;
  scrollbar-width: thin;
  scrollbar-color: #6272a4 transparent;
}

.cursor {
  display: inline-block;
  color: #50fa7b;
}


.footer {
  display: flex;
  justify-content: space-between;
  font-size: 0.7rem;
}

.status {
  color: #6272a4;
}

.status-streaming {
  color: #50fa7b;
}

.status-paused {
  color: #f1b529;
}

.status-done {
  color: #bd93f9;
}

.bytes {
  color: #6272a4;
}
</style>