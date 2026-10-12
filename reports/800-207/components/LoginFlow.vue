<script setup lang="ts">
import { computed } from 'vue'

// 無密碼登入的循序圖；step 0 顯示第 1 組訊息，之後每次點擊多一組
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })

const actors = [
  { id: 'browser', label: '借用電腦瀏覽器', x: 100, color: 'var(--zt-res)' },
  { id: 'pep', label: 'PEP', x: 340, color: 'var(--zt-pep)' },
  { id: 'kc', label: 'Keycloak', x: 600, color: 'var(--zt-idp)' },
  { id: 'phone', label: '手機 passkey', x: 860, color: 'var(--zt-idp)' },
]
const ax = Object.fromEntries(actors.map(a => [a.id, a.x]))

interface Msg { from: string, to?: string, text: string, g: number, dashed?: boolean }
const msgs: Msg[] = [
  { g: 1, from: 'browser', to: 'pep', text: 'GET /a/（還沒有 session）' },
  { g: 1, from: 'pep', to: 'browser', text: '302 → Keycloak（state · PKCE · nonce）' },
  { g: 2, from: 'browser', to: 'kc', text: 'Username Form：只輸入 alice' },
  { g: 2, from: 'kc', to: 'browser', text: 'WebAuthn challenge · RP ID = zt.cloudowo.com' },
  { g: 3, from: 'browser', to: 'phone', text: '跨裝置：掃 QR code ＋ 藍牙確認在附近', dashed: true },
  { g: 3, from: 'phone', text: '指紋 / 臉部解鎖 → 私鑰簽章' },
  { g: 4, from: 'phone', to: 'kc', text: 'assertion（簽章）', dashed: true },
  { g: 4, from: 'kc', text: '用註冊時存的公鑰驗章 ✓' },
  { g: 4, from: 'kc', to: 'browser', text: '302 /_zt/callback?code&state' },
  { g: 5, from: 'browser', to: 'pep', text: 'callback：state 比對 __Host-zt_login cookie' },
  { g: 5, from: 'pep', to: 'kc', text: '後端通道：code + code_verifier 換 token' },
  { g: 5, from: 'pep', text: '驗 ID token（nonce）與 access token（azp）→ 建立 session' },
]

const TOP = 72
const GAP = 28

const rows = msgs.map((m, i) => {
  const y = TOP + i * GAP
  const x1 = ax[m.from]
  if (!m.to) {
    // 自我訊息：手機在最右側，往左畫
    const dir = m.from === 'phone' ? -1 : 1
    return {
      ...m, y,
      d: `M${x1},${y - 8} h${dir * 28} v16 h${-dir * 26}`,
      tx: x1 + dir * 38, ty: y + 4, anchor: dir > 0 ? 'start' : 'end',
    }
  }
  const x2 = ax[m.to]
  const dir = x2 > x1 ? 1 : -1
  return {
    ...m, y,
    d: `M${x1 + dir * 3},${y} L${x2 - dir * 4},${y}`,
    tx: (x1 + x2) / 2, ty: y - 7, anchor: 'middle',
  }
})

const captions = [
  '',
  '<b>PEP 發起 OIDC 授權碼流程</b>：state 綁定這個瀏覽器、PKCE 防授權碼被攔截、<code>prompt=login</code> 每次都重新驗證',
  '<b>Keycloak 只問帳號</b>：整個流程沒有任何密碼欄位',
  '<b>生物辨識只在手機本地完成</b>：網站收不到指紋，私鑰也從不離開手機',
  '<b>伺服器只存公鑰</b>：簽章綁定網域，資料庫外洩或釣魚網站都拿不到能用的憑證',
  '<b>PEP 建立短效 session</b>：效期取 access token、ID token、15 分鐘三者最短',
]

const shown = computed(() => props.step + 1)
</script>

<template>
  <div>
    <svg viewBox="0 0 960 400" class="zt-svg">
      <defs>
        <marker id="login-arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
          <path d="M0,0 L10,5 L0,10 z" style="fill: currentColor" />
        </marker>
      </defs>

      <!-- 參與者與生命線 -->
      <g v-for="a in actors" :key="a.id">
        <line :x1="a.x" y1="40" :x2="a.x" y2="396" class="lifeline" />
        <rect :x="a.x - 78" y="6" width="156" height="34" rx="8" class="actor" :style="{ '--c': a.color }" />
        <text :x="a.x" y="28" text-anchor="middle" class="actor-label">{{ a.label }}</text>
      </g>

      <!-- 訊息 -->
      <g
        v-for="(r, i) in rows" :key="i"
        class="msg"
        :class="{ shown: r.g <= shown, current: r.g === shown, dashed: r.dashed }"
      >
        <path :d="r.d" :pathLength="r.dashed ? undefined : 1" class="msg-line" marker-end="url(#login-arrow)" />
        <text :x="r.tx" :y="r.ty" :text-anchor="r.anchor" class="msg-text halo">{{ r.text }}</text>
      </g>
    </svg>

    <Transition name="cap" mode="out-in">
      <div :key="shown" class="zt-caption text-center" v-html="captions[shown]" />
    </Transition>
  </div>
</template>

<style scoped>
.lifeline {
  stroke: var(--zt-line);
  stroke-dasharray: 3 4;
  opacity: 0.6;
}

.actor {
  fill: color-mix(in srgb, var(--c) 16%, transparent);
  stroke: var(--c);
  stroke-width: 1.5;
}

.actor-label {
  font-size: 14px;
  font-weight: 700;
}

.msg {
  opacity: 0;
  transform: translateY(6px);
  transition: opacity 0.35s, transform 0.35s;
  color: var(--zt-muted);
}

.msg.shown {
  opacity: 1;
  transform: none;
}

.msg.current {
  color: var(--zt-ctrl);
}

.msg-line {
  fill: none;
  stroke: currentColor;
  stroke-width: 1.8;
  stroke-dasharray: 1;
  stroke-dashoffset: 1;
  transition: stroke-dashoffset 0.6s ease 0.1s;
}

.msg.shown .msg-line {
  stroke-dashoffset: 0;
}

.msg.dashed .msg-line,
.msg.dashed.shown .msg-line {
  stroke-dasharray: 6 5;
  stroke-dashoffset: 0;
}

.msg-text {
  font-size: 12.5px;
  fill: var(--zt-muted) !important;
}

.msg.current .msg-text {
  fill: var(--zt-fg) !important;
  font-weight: 600;
}

.zt-caption :deep(code) {
  font-size: 0.85em;
}

.cap-enter-active,
.cap-leave-active {
  transition: opacity 0.25s, transform 0.25s;
}

.cap-enter-from {
  opacity: 0;
  transform: translateY(6px);
}

.cap-leave-to {
  opacity: 0;
  transform: translateY(-6px);
}
</style>
