---
theme: dracula
transition: slide-left
highlighter: shiki
---

# 🔁 Iterators & Generators

一時停止して再開できる関数

---
src: ./pages/basics.md
---

---
src: ./pages/real-cases.md
---

---
layout: center
---

# ジェネレーターはES2015からずっと待っていました

<div class="resources">
  <a href="https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Generator" target="_blank" class="resource">
    <lucide-link class="inline" /> MDN — Generator
  </a>
  <a href="https://ja.javascript.info/generators" target="_blank" class="resource">
    <lucide-link class="inline" /> javascript.info — Generators
  </a>
  <a href="https://ja.javascript.info/async-iterators-generators" target="_blank" class="resource">
    <lucide-link class="inline" /> javascript.info — Async generators
  </a>
</div>

<style scoped>
.resources {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 1rem;
  margin-top: 2rem;
  font-size: 0.875rem;
}
.resource {
  padding: 0.4rem 0.875rem;
  border-radius: 0.375rem;
  background: #282a36;
  border: 1px solid #44475a;
  text-decoration: none;
  color: #f8f8f2;
  transition: border-color 200ms ease;
}
.resource:hover {
  border-color: #bd93f9;
}
</style>