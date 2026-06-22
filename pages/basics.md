---
layout: default
---

# Some processes don't end all at once

Data arrives in pieces. Users request more on demand.

<div class="my-8">
  <IterativeFlow/>
</div>

<v-clicks>

- **Streaming** — server pushes chunks as they're ready
- **Infinite scroll** — data fetched on demand, piece by piece

</v-clicks>

<div v-click class="mt-12">

> If you've struggled to implement these cleanly — generators might be the missing piece.

</div>


---
layout: two-cols-header
gap: 2
---

# The straightforward implementation

::left::

```js
async function streamText(url) {
    const res = await fetch(url)
    const reader = res.body.getReader()

    while (true) {
        const {done, value} = await reader.read()
        if (done) break
        setText(prev => prev + decode(value))
    }
}
```

::right::

<v-click>

**Problems**

* Fetching and consuming are **tightly coupled** — change one, touch both

</v-click>
<v-click>

* Impossible to **test in isolation** — you can't unit test streaming logic without triggering a re-render

</v-click>

<style>
.two-cols-header { column-gap: 2rem; }
</style>

---
layout: two-cols-header
---

# A step in the right direction

::left::

```js
async function streamText(url, onChunk) {
    const res = await fetch(url)
    const reader = res.body.getReader()

    while (true) {
        const {done, value} = await reader.read()
        if (done) break
        onChunk(decode(value))
    }
}
```

::right::

<v-click>

✅ Consumption is **decoupled** — caller decides what to do with each chunk

</v-click>
<v-click>

❌ **Push-based** only — callbacks and `.then()` were replaced by `await` for a reason

```js
// push-based — hard to follow
fetch(url).then(res => {
    // otherFunction() will be called before this callback
})
otherFunction()

// sequential — clear and intuitive
const res = await fetch(url)
// Some additional lines of code
otherFunction()
```

</v-click>

<style>
.two-cols-header { column-gap: 2rem; }
</style>

---
---

# What if consuming chunks looked like this?

```js
for await (const chunk of streamChunks(url)) {
    // it doesnt matter to use how this chunk was acquired, 
    // we get the results as soon as its ready
    render(chunk)
}
```

<v-clicks>

- **Sequential** — reads top to bottom, no callbacks
- **Pull-based** — consumer drives the pace, not the producer
- **Decoupled** — `streamChunks` knows nothing about `render`
- **Testable** — iterate the generator directly in any test

</v-clicks>
<v-click>

This is exactly what generators enable. Let's see how.

</v-click>


---
layout: two-cols-header
---

# Iterators & Iterables

::left::

An **iterator** is an object that produces values one at a time.

- has a `next()` method
- `next()` returns `{ value, done }`

<div class="my-8">
<IteratorDemo />
</div>


::right::

An **iterable** is an object that exposes an iterator.

- has a `[Symbol.iterator]()` method
- that method returns an **iterator**

Anything that works with `for...of` or spread is an iterable

* `Array`
* `String`
* `Map`
* `Set`

<!-- 
Live demo: Array[Symbol.iterator] is non-enumerable but accessible.

const arr = [1, 2, 3]
const iter = arr[Symbol.iterator]()
iter.next()
-->

---
layout: two-cols-header
---

# Building an iterator manually

::left::

Let's implement `range(start, end)` — a sequence of numbers from start to end, generated on demand.

> Same idea as Python's `range()`.

```js
function range(start, end) {
    return {
        [Symbol.iterator]() {
            let current = start
            return {
                next() {
                    return current < end
                        ? {value: current++, done: false}
                        : {value: undefined, done: true}
                }
            }
        }
    }
}
```

::right::

<v-click>

```js
// usage
for (const n of range(1, 5)) {
    console.log(n) // 1, 2, 3, 4, 5
}
```

</v-click>

<v-click>

* Using array

```js
const arr = Array.from({length: 1000000})
// ☝️ 1M slots allocated in memory
```

</v-click>


<v-click>

* Using iterator

```js
const iter = range(1, 1000000)
// ☝️ no numbers exist yet
// each value generated on demand
```

</v-click>

<style>
.two-cols-header { column-gap: 2rem; }
</style>

---
layout: two-cols-header
---

# Iterable iterator & Generator

::left::

An **iterable iterator** is both at once

<v-click>

> `[Symbol.iterator]()` returns `this`

```js
function range(start, end) {
  let current = start
    
  return {
    [Symbol.iterator]() { return this },
      
    next() {
      return current <= end
        ? { value: current++, done: false }
        : { value: undefined, done: true }
    }
  }
}
```

</v-click>

::right::

<v-click>

A **generator** is an iterable iterator — built automatically by `function*`

```js
function* range(start, end) {
  for (let i = start; i <= end; i++) {
    yield i
  }
}
```
</v-click>

<v-click>

`yield` pauses execution and returns the current value — the generator resumes from the same point on the next `next()` call.

</v-click>

<v-clicks>

- only valid inside `function*`
- replaces the manual `{ value, done }` shape
- `next()` and `{ value, done }` are still there — just hidden under the hood

</v-clicks>

<style>
.two-cols-header { column-gap: 2rem; }
</style>

