<template>
  <div class="wrapper">
    <div class="iterator-box" :class="{ done: isDone }">
      <span class="label">iterator</span>
      <div class="sequence">
        <span
            v-for="(item, i) in items"
            :key="i"
            class="item"
            :class="{ active: i === current - 1, consumed: i < current - 1 }"
        >{{ item }}</span>
      </div>
      <div class="result" :class="{ visible: current > 0 }">
        <span class="brace">&#123;</span>
        <span class="field">value: <span class="val">{{ displayValue }}</span></span>
        <span class="sep">,</span>
        <span class="field">done: <span class="val" :class="{ 'is-done': isDone }">{{ isDone }}</span></span>
        <span class="brace">&#125;</span>
      </div>
    </div>
    <button class="next-btn" :disabled="isDone" @click="advance">
      {{ isDone ? 'done' : 'next()' }}
    </button>
  </div>
</template>

<script setup>
import {ref, computed} from 'vue'

const items = ['1️⃣', '2️⃣', '3️⃣', '4️⃣', '5️⃣']
const current = ref(0)

const isDone = computed(() => current.value > items.length)
const displayValue = computed(() => {
  if (current.value === 0) return '—'
  if (current.value > items.length) return 'undefined'
  return `"${items[current.value - 1]}"`
})

function advance() {
  if (!isDone.value) current.value++
}
</script>

<style scoped>
.wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
  font-family: ui-monospace, monospace;
}

.iterator-box {
  position: relative;
  border: 1.5px solid var(--slidev-theme-accents-teal, #6272a4);
  border-radius: 10px;
  padding: 28px 32px 20px;
  min-width: 340px;
  background: rgba(255, 255, 255, 0.04);
  transition: border-color 0.3s;
}

.iterator-box.done {
  border-color: #50fa7b;
}

.label {
  position: absolute;
  top: -10px;
  left: 16px;
  background: #282a36;
  padding: 0 8px;
  font-size: 11px;
  color: #6272a4;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

.sequence {
  display: flex;
  gap: 12px;
  justify-content: center;
  margin-bottom: 20px;
  font-size: 1.8rem;
}

.item {
  transition: all 0.2s;
  opacity: 0.25;
  transform: scale(0.85);
}

.item.active {
  opacity: 1;
  transform: scale(1.25);
}

.item.consumed {
  opacity: 0.12;
  transform: scale(0.8);
}

.result {
  display: flex;
  gap: 6px;
  align-items: center;
  justify-content: center;
  font-size: 13px;
  color: #f8f8f2;
  opacity: 0;
  transition: opacity 0.2s;
}

.result.visible {
  opacity: 1;
}

.brace {
  color: #f8f8f2;
}

.field {
  color: #8be9fd;
}

.sep {
  color: #f8f8f2;
}

.val {
  color: #f1fa8c;
}

.val.is-done {
  color: #50fa7b;
}

.next-btn {
  font-family: ui-monospace, monospace;
  font-size: 14px;
  padding: 8px 28px;
  background: transparent;
  color: #f8f8f2;
  border: 1.5px solid #6272a4;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
}

.next-btn:hover:not(:disabled) {
  background: #44475a;
  border-color: #bd93f9;
}

.next-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
  border-color: #50fa7b;
  color: #50fa7b;
}
</style>