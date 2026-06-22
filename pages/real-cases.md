---
layout: two-cols-header
---

# Generators in practice — PRNG

::left::

<a href="https://en.wikipedia.org/wiki/Pseudorandom_number_generator" target="_blank">Pseudorandom Number Generator (PRNG) ↗</a>

A deterministic algorithm that produces a sequence of numbers that appear random — but given the same seed, always produces the same sequence.

**Used in:** mocking data, procedural world generation, reproducible test fixtures.

```js
function* seededUsers(seed) {
  let s = seed
  while (true) {
    s = (s * 1664525 + 1013904223) & 0x7fffffff
    const first = firstNames[s % firstNames.length]
    s = (s * 1664525 + 1013904223) & 0x7fffffff
    const last = lastNames[s % lastNames.length]
    yield `${first} ${last}`
  }
}
```

::right::

<UserGenerator />

<style>
.two-cols-header { column-gap: 2rem; }
</style>

<!--
LCG — Linear Congruential Generator
-->

---
layout: default
---

# Why haven't you heard of this?

<v-click>

The core principle of generators is simple: **return the next item on demand**. Async works too — just wait for the next one when it's ready.

This step-based, on-demand mechanism maps naturally to backend problems — cursor navigation, partial file reading, 
database streaming. Backend developers deal with these quite frequently, which is why generators feel familiar there.

</v-click>

<v-click>

On the frontend, there are a few common use cases:

</v-click>

<v-clicks>

- **Infinite scroll** — load the next page when the user is ready
- **Real-time streaming** — consume chunks as they arrive from the server

</v-clicks>


<v-click>

Both are pull-based, step-driven flows. Generators are a natural fit — and that's exactly what we'll build next.

</v-click>


---
layout: default
---

# Async generators

When the next value isn't immediately available — it needs to be fetched, waited for, or streamed — a regular generator falls short.

<strong>Async generators</strong> let you `await` between yields, making asynchronous sequences feel just as clean as synchronous ones.

<div class="mt-8">
  <AsyncGeneratorFlow />
</div>

---
layout: two-cols-header
---

# From sync to async

::left::

```js
function* range(start, end) {
  for (let i = start; i <= end; i++) {
    yield i
  }
}

for (const value of range(1, 5)) {
  console.log(value)
}
```

::right::

```js {*|1|3|8|*}
async function* asyncRange(start, end) {
  for (let i = start; i <= end; i++) {
    await new Promise(r => setTimeout(r, 1000))
    yield i
  }
}

for await (const value of asyncRange(1, 5)) {
  console.log(value)
}
```

<v-click>

main differences

</v-click>

<v-clicks>

- add `async` to `function*`
- `await` inside the body
- `for await...of` to iterate

</v-clicks>

<style>
.two-cols-header { column-gap: 2rem; }
</style>

---
---

# Async iterables gochas

Spread and destructuring expect `Symbol.iterator` — async iterables use `Symbol.asyncIterator` instead.

```js
async function* asyncRange(start, end) {
  for (let i = start; i <= end; i++) {
    yield i
  }
}

// ❌ won't work
const values = [...asyncRange(1, 5)] // TypeError: range(...) is not iterable

// ✅ only way to consume
for await (const value of asyncRange(1, 5)) {
  console.log(value)
}

// ✅ also valid
const asyncGen = asyncRange(1, 5)
const { value, done } = await asyncGen.next()
```

---
layout: two-cols-header
---

# Infinite scroll

::left::

An async generator owns the cursor internally — the consumer just calls `next()`.

```js
async function* fetchCommits(repo) {
  let url = `https://api.github.com/repos/${repo}/commits`
  while (url) {
    const res = await fetch(url)
    const data = await res.json()
    url = getNextPage(res.headers)
    for (const commit of data) yield commit
  }
}
```

No page counter. No external state. The generator resumes exactly where it left off on each `load more` click.

::right::

<GithubCommits />

<div class="mt-2 text-xs opacity-40">

⚠️ GitHub API is limited to 60 requests/hour for unauthenticated users

</div>

<style>
.two-cols-header { column-gap: 2rem; }
</style>


---
layout: two-cols-header
---

# Streaming

::left::

An async generator wraps the `ReadableStream` API — yielding decoded chunks one at a time.

```js
async function* streamChunks(url) {
  const res = await fetch(url)
  const reader = res.body.getReader()
  const decoder = new TextDecoder()

  while (true) {
    const { done, value } = await reader.read()
    if (done) break
    yield decoder.decode(value, { stream: true })
  }
}

for await (const chunk of streamChunks(url)) {
  render(chunk)
}
```

The consumer calls `next()` at its own pace — pause anytime, resume exactly where it left off.

::right::

<StreamingDemo />

<style>
.two-cols-header { column-gap: 2rem; }
</style>