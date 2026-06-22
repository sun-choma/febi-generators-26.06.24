<template>
  <div class="wrapper">
    <div class="controls">
      <input
          v-model="repoInput"
          type="text"
          placeholder="owner/repo"
          class="repo-input"
          :disabled="loading"
      />
      <button class="btn-fetch" :disabled="loading" @click="fetchRepo">
        fetch repo
      </button>
    </div>

    <div class="list-wrapper">
      <div class="list">
        <div v-if="commits.length === 0 && !loading && !error" class="empty">
          — enter a repo and press "fetch repo" —
        </div>
        <div v-for="(commit, i) in commits" :key="i" class="commit-row">
          <div class="commit-top">
            <img v-if="commit.avatar" :src="commit.avatar" class="avatar" />
            <div class="commit-body">
              <a :href="commit.url" target="_blank" class="commit-msg">{{ commit.message }}</a>
              <span class="commit-meta">{{ commit.author }} · {{ commit.date }}</span>
            </div>
          </div>
        </div>
        <div class="load-more-row">
          <span v-if="error" class="error">⚠️ {{ error }}</span>
          <span v-else-if="loading" class="loading-indicator">loading...</span>
          <span v-else-if="exhausted" class="exhausted">— end of history —</span>
          <a v-else-if="commits.length > 0" href="#" class="load-more-link" @click.prevent="loadMore">
            load more
          </a>
        </div>
      </div>
    </div>

    <div class="footer">{{ commits.length }} commits loaded</div>
  </div>
</template>

<script setup>
import { ref, nextTick } from 'vue'

const repoInput = ref('vuejs/core')
const commits = ref([])
const loading = ref(false)
const exhausted = ref(false)
const error = ref(null)

let gen = null

async function* fetchCommits(repo) {
  let url = `https://api.github.com/repos/${repo}/commits?per_page=10`
  while (url) {
    const res = await fetch(url, {
      headers: { 'Accept': 'application/vnd.github.v3+json' }
    })
    if (!res.ok) throw new Error(`GitHub API error: ${res.status}`)
    const data = await res.json()
    const linkHeader = res.headers.get('Link') || ''
    const match = linkHeader.match(/<([^>]+)>;\s*rel="next"/)
    url = match ? match[1] : null
    for (const commit of data) {
      yield {
        message: commit.commit.message.split('\n')[0],
        author: commit.commit.author.name,
        date: new Date(commit.commit.author.date).toLocaleDateString(),
        avatar: commit.author?.avatar_url ?? null,
        url: commit.html_url
      }
    }
  }
}

async function fetchRepo() {
  commits.value = []
  exhausted.value = false
  error.value = null
  gen = fetchCommits(repoInput.value.trim())
  await loadMore()
}

async function loadMore() {
  if (loading.value || !gen || exhausted.value) return
  loading.value = true

  try {
    let loaded = 0
    while (loaded < 10) {
      const { value, done } = await gen.next()
      if (done) {
        exhausted.value = true
        break
      }
      commits.value.push(value)
      loaded++
    }
    await nextTick()
  } catch (e) {
    error.value = e.message
    exhausted.value = true
  } finally {
    loading.value = false
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
}

.repo-input {
  flex: 1;
  font-family: ui-monospace, monospace;
  font-size: 0.78rem;
  padding: 6px 10px;
  background: #21222c;
  border: 1.5px solid #6272a4;
  border-radius: 4px;
  color: #f8f8f2;
  outline: none;
  transition: border-color 0.2s;
}

.repo-input:focus {
  border-color: #bd93f9;
}

.repo-input:disabled {
  opacity: 0.5;
}

.btn-fetch {
  font-family: ui-monospace, monospace;
  font-size: 0.78rem;
  padding: 6px 16px;
  background: transparent;
  border: 1.5px solid #bd93f9;
  border-radius: 4px;
  color: #bd93f9;
  cursor: pointer;
  transition: all 0.2s;
  white-space: nowrap;
}

.btn-fetch:hover:not(:disabled) {
  background: #44475a;
}

.btn-fetch:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.list-wrapper {
  border: 1.5px solid #6272a4;
  border-radius: 8px;
  height: 220px;
  overflow-y: auto;
  background: #21222c;
  scrollbar-width: thin;
  scrollbar-color: #6272a4 transparent;
}

.list {
  padding: 8px 12px;
  display: flex;
  flex-direction: column;
}

.empty {
  font-size: 0.72rem;
  color: #6272a4;
  padding: 4px 0;
}

.commit-row {
  display: flex;
  flex-direction: column;
  gap: 2px;
  padding: 7px 0;
  border-bottom: 1px solid #44475a;
}

.commit-row:last-of-type {
  border-bottom: none;
}

.commit-top {
  display: flex;
  gap: 10px;
  align-items: flex-start;
}

.avatar {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  flex-shrink: 0;
  margin-top: 2px;
}

.commit-body {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.commit-msg {
  font-size: 0.78rem;
  color: #bd93f9;
  text-decoration: none;
  border: none;
  max-height: 2.25rem;
  overflow: hidden;
}

.commit-msg:hover {
  text-decoration: underline;
  border: none;
}

.commit-meta {
  font-size: 0.68rem;
  color: #6272a4;
}

.load-more-row {
  padding: 10px 0;
  text-align: center;
}

.load-more-link {
  font-size: 0.75rem;
  color: #bd93f9;
  text-decoration: none;
  cursor: pointer;
  border: none;
}

.load-more-link:hover {
  text-decoration: underline;
  border: none;
}

.loading-indicator {
  font-size: 0.75rem;
  color: #6272a4;
}

.exhausted {
  font-size: 0.75rem;
  color: #6272a4;
}

.error {
  font-size: 0.75rem;
  color: #ff5555;
}

.footer {
  font-size: 0.7rem;
  color: #6272a4;
  text-align: right;
}
</style>