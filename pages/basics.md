---
layout: default
---

# 一部のプロセスは一度に終わらない

データは少しずつ届く。ユーザーは必要なときに追加のデータをリクエストする。

<div class="my-8">
  <IterativeFlow/>
</div>

<v-clicks>

- **ストリーミング**。サーバーが準備でき次第、チャンクをプッシュする
- **無限スクロール**。データをオンデマンドで少しずつ取得する

</v-clicks>

<div v-click class="mt-12">

> これらをクリーンに実装するのに苦労したことがあるなら、ジェネレーターが解決策かもしれません。

</div>


---
layout: two-cols-header
gap: 2
---

# シンプルな実装

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

**問題点**

* フェッチと消費が**密結合**。片方を変えると、もう片方も変える必要がある

</v-click>
<v-click>

* **単体テストが不可能**。レンダリングをトリガーせずにストリーミングロジックをテストできない

</v-click>

<style>
.two-cols-header { column-gap: 2rem; }
</style>

---
layout: two-cols-header
---

# 改善された実装

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

✅ 消費ロジックが**分離された**。呼び出し側が各チャンクの処理を決定できる

</v-click>
<v-click>

❌ **プッシュ型**のまま。コールバックと`.then()`が`await`に置き換えられた理由がある

```js
// プッシュ型。流れを追いにくい
fetch(url).then(res => {
    // otherFunction() はこのコールバックより先に呼ばれる
})
otherFunction()

// 順次実行。明快で直感的
const res = await fetch(url)
// 追加のコード
otherFunction()
```

</v-click>

<style>
.two-cols-header { column-gap: 2rem; }
</style>

---
---

# チャンクの消費がこう書けたら？

```js
for await (const chunk of streamChunks(url)) {
    // チャンクがどう取得されたかは関係ない
    // 準備でき次第、結果を受け取れる
    render(chunk)
}
```

<v-clicks>

- **順次実行**。上から下へ読め、コールバック不要
- **プル型**。プロデューサーではなく、コンシューマーがペースをコントロールする
- **疎結合**。`streamChunks`は`render`について何も知らない
- **テスト可能**。どんなテストでもジェネレーターを直接イテレートできる

</v-clicks>
<v-click>

これがまさにジェネレーターが実現することです。実装を見ていきましょう。

</v-click>


---
layout: two-cols-header
---

# イテレーターとイテラブル

::left::

**イテレーター**とは、値を一つずつ生成するオブジェクトです。

- `next()`メソッドを持つ
- `next()`は`{ value, done }`を返す

<div class="my-8">
<IteratorDemo />
</div>


::right::

<v-click>

**イテラブル**とは、イテレーターを公開するオブジェクトです。

- `[Symbol.iterator]()`メソッドを持つ
- そのメソッドが**イテレーター**を返す

`for...of`やスプレッド構文で使えるものは全てイテラブル

</v-click>

<v-clicks>

* `Array`
* `String`
* `Map`
* `Set`

</v-clicks>

<!--
Live demo: Array[Symbol.iterator] は列挙不可能だがアクセス可能。

const arr = [1, 2, 3]
const iter = arr[Symbol.iterator]()
iter.next()
-->

---
layout: two-cols-header
---

# イテラブルを手動で実装する

::left::

`range(start, end)`を実装してみましょう。startからendまでの数値をオンデマンドで生成するシーケンスです。

> Pythonの`range()`と同じ考え方です。

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
// 使い方
for (const n of range(1, 5)) {
    console.log(n) // 1, 2, 3, 4
}
```

</v-click>

<v-click>

* 配列を使う場合

```js
const arr = Array.from({length: 1_000_000})
// ☝️ 100万スロットがメモリに確保される
```

</v-click>

<!--
ES2021
-->


<v-click>

* イテレーターを使う場合

```js
const iter = range(1, 1_000_000)
// ☝️ まだ数値は存在しない
// 値はオンデマンドで生成される
```

</v-click>

<style>
.two-cols-header { column-gap: 2rem; }
</style>

---
layout: two-cols-header
---

# イテラブルイテレーターとジェネレーター

::left::

**イテラブルイテレーター**は両方を兼ねるオブジェクトです

<v-click>

> `[Symbol.iterator]()`が`this`を返す

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

**ジェネレーター**はイテラブルイテレーターを自動で生成します。`function*`で定義します。

```js
function* range(start, end) {
  for (let i = start; i <= end; i++) {
    yield i
  }
}
```
</v-click>

<v-click>

`yield`は実行を一時停止し、現在の値を返します。次の`next()`呼び出しで同じ箇所から再開します。

</v-click>

<v-clicks>

- `function*`の内部でのみ有効
- 手動の`{ value, done }`の記述を置き換える
- `next()`と`{ value, done }`は引き続き存在する。ただし内部に隠されている

</v-clicks>

<style>
.two-cols-header { column-gap: 2rem; }
</style>