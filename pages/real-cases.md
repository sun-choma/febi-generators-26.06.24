---
layout: two-cols-header
---

# ジェネレーターの実践例

::left::

<a href="https://ja.wikipedia.org/wiki/%E6%93%AC%E4%BC%BC%E4%B9%B1%E6%95%B0" target="_blank">擬似乱数生成器（PRNG） ↗</a>

同じシードを与えると常に同じ数列を生成する、決定論的なアルゴリズムです。

**主な用途：** モックデータの生成、プロシージャルワールド生成、再現性のあるテストデータ。

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
LCG — Linear Congruential Generator（線形合同法）
-->

---
layout: default
---

# なぜ聞いたことがないのか？

<v-click>

ジェネレーターの核心はシンプルです。**オンデマンドで次のアイテムを返す**。非同期も対応しています。準備ができたら次のアイテムを待つだけです。

このステップ駆動・オンデマンドの仕組みは、カーソルナビゲーション、部分的なファイル読み込み、データベースストリーミングなど、バックエンドの問題に自然にマッチします。バックエンド開発者が日常的に扱うため、ジェネレーターはそちらでは馴染みのある存在です。

</v-click>

<v-click>

フロントエンドでも、いくつかの場面で活躍します。

</v-click>

<v-clicks>

- **無限スクロール**。ユーザーの操作に合わせて次のページを読み込む
- **リアルタイムストリーミング**。サーバーからチャンクが届き次第、消費する

</v-clicks>

<v-click>

どちらもプル型・ステップ駆動のフローです。ジェネレーターが自然にフィットします。次はこれを実際に構築していきましょう。

</v-click>


---
layout: default
---

# 非同期ジェネレーター

次の値がすぐに取得できない場合、つまりフェッチや待機、ストリーミングが必要な場合、通常のジェネレーターでは対応できません。

**非同期ジェネレーター**を使うと`yield`の間で`await`が使えるため、非同期なシーケンスも同期処理と同じくらいクリーンに書けます。

<div class="mt-8">
  <AsyncGeneratorFlow />
</div>

---
layout: two-cols-header
---

# 同期から非同期へ

::left::

```js
function* range(start, end) {
    for (let i = start; i < end; i++) {
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
    for (let i = start; i < end; i++) {
        await new Promise(r => setTimeout(r, 1000))
        yield i
    }
}

for await (const value of asyncRange(1, 5)) {
    console.log(value)
}
```

<v-click>

主な変更点

</v-click>

<v-clicks>

- `function*`に`async`を追加する
- 関数本体で`await`が使える
- イテレートには`for await...of`を使う

</v-clicks>

<style>
.two-cols-header { column-gap: 2rem; }
</style>

---
---

# 非同期イテラブルの注意点

スプレッド構文やデストラクチャリングは`Symbol.iterator`を期待しますが、非同期イテラブルは`Symbol.asyncIterator`を使います。

```js
async function* asyncRange(start, end) {
    for (let i = start; i < end; i++) {
        yield i
    }
}

// ❌ 動作しない
const values = [...asyncRange(1, 5)] // TypeError: range(...) is not iterable

// ✅ 標準的な消費方法
for await (const value of asyncRange(1, 5)) {
    console.log(value)
}

// ✅ こちらも有効
const asyncGen = asyncRange(1, 5)
const {value, done} = await asyncGen.next()
```

---
layout: two-cols-header
---

# 無限スクロール

::left::

非同期ジェネレーターがカーソルを内部で管理します。コンシューマーは`next()`を呼ぶだけです。

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

ページカウンターなし。外部ステートなし。「もっと読む」クリックのたびに、ジェネレーターは中断した箇所から正確に再開します。

::right::

<GithubCommits />

<div class="mt-2 text-xs opacity-40">

⚠️ GitHub APIは未認証ユーザーに対して1時間あたり60リクエストの制限があります

</div>

<style>
.two-cols-header { column-gap: 2rem; }
</style>


---
layout: two-cols-header
---

# ストリーミング

::left::

非同期ジェネレーターが`ReadableStream` APIをラップし、チャンクを一つずつyieldします。

```js
async function* streamChunks(url) {
    const res = await fetch(url)
    const reader = res.body.getReader()
    const decoder = new TextDecoder()

    while (true) {
        const {done, value} = await reader.read()
        if (done) break
        yield decoder.decode(value, {stream: true})
    }
}

for await (const chunk of streamChunks(url)) {
    render(chunk)
}
```

コンシューマーは自分のペースで`next()`を呼びます。いつでも一時停止でき、再開できます。

::right::

<StreamingDemo />

> ⚠️ ストリーミング速度はデモのため意図的に遅くしています

<style>
.two-cols-header { column-gap: 2rem; }
</style>