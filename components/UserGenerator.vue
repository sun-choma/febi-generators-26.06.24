<template>
  <div class="wrapper">
    <div class="controls">
      <span class="seed-label">seed:</span>
      <input v-model.number="seedInput" type="number" class="seed-input" @change="reset" />
      <button class="btn btn-primary" @click="generateNext">next()</button>
      <button class="btn btn-reset" @click="reset">reset</button>
    </div>

    <div class="list-wrapper" ref="listWrapper">
      <div class="list">
        <div v-if="users.length === 0" class="empty">— press next() to generate —</div>
        <div v-for="(user, i) in users" :key="i" class="row">
          <span class="index">{{ i + 1 }}</span>
          <span class="name">{{ user }}</span>
        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, nextTick } from 'vue'

const firstNames = ['Haruto','Yuki','Sota','Aoi','Ren','Hina','Kaito','Sakura','Yuna','Riku','Mio','Sora','Akira','Nana','Kento']
const lastNames = ['Sato','Suzuki','Tanaka','Watanabe','Ito','Yamamoto','Nakamura','Kobayashi','Kato','Yoshida','Yamada','Sasaki','Yamaguchi','Matsumoto','Inoue']

const seedInput = ref(42)
const users = ref([])
const listWrapper = ref(null)

let state = 42
let gen = null

function lcg(s) {
  return (s * 1664525 + 1013904223) & 0x7fffffff
}

function* userGenerator(seed) {
  let s = seed
  while (true) {
    s = lcg(s)
    const first = firstNames[s % firstNames.length]
    s = lcg(s)
    const last = lastNames[s % lastNames.length]
    yield `${first} ${last}`
  }
}

function reset() {
  users.value = []
  gen = userGenerator(seedInput.value)
}

async function generateNext() {
  if (!gen) gen = userGenerator(seedInput.value)
  users.value.push(gen.next().value)
  await nextTick()
  if (listWrapper.value) {
    listWrapper.value.scrollTop = listWrapper.value.scrollHeight
  }
}

reset()
</script>

<style scoped>
.wrapper {
  font-family: ui-monospace, monospace;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.controls {
  display: flex;
  align-items: center;
  gap: 8px;
}

.seed-label {
  font-size: 0.75rem;
  color: #6272a4;
  white-space: nowrap;
}

.seed-input {
  width: 72px;
  background: #21222c;
  border: 1.5px solid #6272a4;
  border-radius: 4px;
  color: #f8f8f2;
  font-family: ui-monospace, monospace;
  font-size: 0.8rem;
  padding: 4px 8px;
  outline: none;
}

.seed-input:focus {
  border-color: #bd93f9;
}

.btn {
  font-family: ui-monospace, monospace;
  font-size: 0.8rem;
  padding: 5px 14px;
  border-radius: 4px;
  cursor: pointer;
  border: 1.5px solid;
  transition: all 0.2s;
}

.btn-primary {
  background: transparent;
  border-color: #bd93f9;
  color: #bd93f9;
  flex: 1;
}

.btn-primary:hover {
  background: #44475a;
}

.btn-reset {
  background: transparent;
  border-color: #6272a4;
  color: #6272a4;
  flex: 1;
}

.btn-reset:hover {
  background: #44475a;
  border-color: #f8f8f2;
  color: #f8f8f2;
}

.list-wrapper {
  border: 1.5px solid #6272a4;
  border-radius: 8px;
  height: 200px;
  overflow-y: auto;
  background: #21222c;
  scrollbar-width: thin;
  scrollbar-color: #6272a4 transparent;
}

.list {
  padding: 8px 12px;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.empty {
  font-size: 0.72rem;
  color: #6272a4;
  padding: 4px 0;
}

.row {
  display: flex;
  gap: 12px;
  align-items: center;
  padding: 4px 0;
  border-bottom: 1px solid #44475a;
  font-size: 0.82rem;
}

.row:last-child {
  border-bottom: none;
}

.index {
  color: #6272a4;
  min-width: 20px;
  text-align: right;
  font-size: 0.72rem;
}

.name {
  color: #f8f8f2;
}

.footer {
  font-size: 0.7rem;
  color: #6272a4;
  text-align: right;
}
</style>