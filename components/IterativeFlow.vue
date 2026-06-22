<template>
  <div class="flow-wrapper">
    <div class="flow-diagram" ref="diagram">
      <div class="step c-amber">
        <span class="label">ユーザー操作</span>
        <span class="sub">スクロール / リクエスト</span>
      </div>
      <div class="arrow">→</div>
      <div class="step c-purple">
        <span class="label">チャンク取得</span>
        <span class="sub">サーバー / データ</span>
      </div>
      <div class="arrow">→</div>
      <div class="step c-teal">
        <span class="label">表示</span>
        <span class="sub">ユーザーへレンダリング</span>
      </div>
    </div>

    <svg class="loop-arrow" viewBox="0 0 500 40" xmlns="http://www.w3.org/2000/svg">
      <defs>
        <marker id="loop-head" viewBox="0 0 10 10" refX="8" refY="5"
                markerWidth="6" markerHeight="6" orient="auto-start-reverse">
          <path d="M2 1L8 5L2 9" fill="none" stroke="#6272a4" stroke-width="1.5"
                stroke-linecap="round" stroke-linejoin="round"/>
        </marker>
      </defs>
      <path
          d="M 470 4 Q 470 36 250 36 Q 30 36 30 4"
          fill="none"
          stroke="#6272a4"
          stroke-width="1.5"
          stroke-dasharray="4 3"
          marker-end="url(#loop-head)"
      />
      <text x="250" y="32" text-anchor="middle" fill="#6272a4" opacity="0.7"
            font-family="ui-monospace, monospace">繰り返し
      </text>
    </svg>
  </div>
</template>

<style scoped>
.flow-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0;
}

.flow-diagram {
  display: flex;
  align-items: center;
  gap: 8px;
}

.step {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 12px 20px;
  border-radius: 8px;
  min-width: 130px;
  border: 1px solid;
}

.c-amber {
  background: #3d2e0e;
  border-color: #f1b529;
  color: #f1b529;
}

.c-purple {
  background: #2a1f4a;
  border-color: #bd93f9;
  color: #bd93f9;
}

.c-teal {
  background: #0e3130;
  border-color: #50fa7b;
  color: #50fa7b;
}

.label {
  font-size: 0.95rem;
  font-weight: 600;
}

.sub {
  font-size: 0.72rem;
  opacity: 0.65;
  margin-top: 2px;
}

.arrow {
  font-size: 1.4rem;
  opacity: 0.5;
  color: #f8f8f2;
}

.loop-arrow {
  width: 500px;
  height: 40px;
  margin-top: -2px;
}
</style>