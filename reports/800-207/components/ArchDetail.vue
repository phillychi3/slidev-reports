<script setup lang="ts">
import { computed } from 'vue'

// 詳細部署圖：跨在網段邊界上的元件同時接兩個網段。
// step 1–3 依類別點亮連線並標出傳遞內容，step 4 依連線保護等級上色。
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })

type Prot = 'ok' | 'sig' | 'none'

const nodes = [
  { id: 'browser', x: 16, y: 30, w: 126, h: 52, label: '借用電腦', sub: '瀏覽器 · 無裝置憑證', color: 'var(--zt-res)' },
  { id: 'phone', x: 16, y: 128, w: 126, h: 52, label: '手機', sub: 'passkey', color: 'var(--zt-idp)' },
  { id: 'admin', x: 16, y: 318, w: 126, h: 52, label: '伺服器管理者', sub: '模擬 CDM / TI', color: 'var(--zt-pip)' },
  { id: 'caddy', x: 195, y: 34, w: 130, h: 52, label: 'Caddy', sub: ':443 公開 HTTPS', color: 'var(--zt-res)' },
  { id: 'keycloak', x: 390, y: 126, w: 140, h: 54, label: 'Keycloak', sub: ':8080 passkey', tag: 'IdP', color: 'var(--zt-idp)' },
  { id: 'pep', x: 605, y: 126, w: 140, h: 54, label: 'PEP', sub: ':8443 Go proxy', tag: 'PEP', color: 'var(--zt-pep)' },
  { id: 'ca', x: 195, y: 206, w: 130, h: 46, label: 'step-ca', sub: 'PEP 伺服器憑證', tag: 'PKI', color: 'var(--zt-pip)' },
  { id: 'pe', x: 195, y: 296, w: 125, h: 54, label: 'PE', sub: ':8081 /decide', tag: 'PE', color: 'var(--zt-ctrl)' },
  { id: 'redis', x: 395, y: 296, w: 130, h: 54, label: 'Redis', sub: 'PIP · session · 事件', tag: 'PIP', color: 'var(--zt-pip)' },
  { id: 'pa', x: 615, y: 296, w: 120, h: 54, label: 'PA', sub: ':8082 /access', tag: 'PA', color: 'var(--zt-ctrl)' },
  { id: 'res-a', x: 858, y: 30, w: 120, h: 50, label: '資源 a', sub: '一般', color: 'var(--zt-res)' },
  { id: 'res-b', x: 858, y: 170, w: 120, h: 50, label: '資源 b', sub: '敏感', color: 'var(--zt-res)' },
  { id: 'res-c', x: 858, y: 310, w: 120, h: 50, label: '資源 c', sub: '管理', color: 'var(--zt-res)' },
]

interface Edge {
  id: string
  d: string
  on: number[]
  prot: Prot
  label?: string
  lx?: number
  ly?: number
  anchor?: string
  both?: boolean
  dashed?: boolean
  blocked?: boolean
  noProt?: boolean // 不是網路連線（docker exec），保護等級那一步不顯示
}

const edges: Edge[] = [
  // 1 公開入口
  { id: 'b-caddy', d: 'M142,58 L193,60', on: [1], prot: 'ok', label: 'HTTPS 公開 CA', lx: 147, ly: 24 },
  { id: 'ph-b', d: 'M44,126 L44,84', on: [1], prot: 'ok', both: true, dashed: true, label: 'QR + BLE', lx: 52, ly: 109 },
  { id: 'caddy-pep', d: 'M325,52 C480,30 640,40 668,124', on: [1], prot: 'ok', label: 'HTTPS（step-ca，驗 SAN=pep）· 不轉送用戶端憑證', lx: 352, ly: 28 },
  { id: 'caddy-kc', d: 'M300,86 C330,108 400,100 440,124', on: [1], prot: 'none', label: '/auth/*', lx: 352, ly: 118 },
  // 2 控制平面
  { id: 'pep-kc', d: 'M605,168 L532,168', on: [2], prot: 'sig', label: 'token / JWKS', lx: 568, ly: 196, anchor: 'middle' },
  { id: 'pep-pa', d: 'M675,180 L675,294', on: [2], prot: 'none', label: 'POST /access', lx: 682, ly: 240 },
  { id: 'redis-pep', d: 'M480,294 C500,240 600,220 640,182', on: [2], prot: 'none', label: 'SUBSCRIBE pep:revoke', lx: 470, ly: 228 },
  { id: 'pa-redis', d: 'M615,323 L527,323', on: [2], prot: 'none', label: 'SET session · SADD grants\nPUBLISH · XADD', lx: 571, ly: 368, anchor: 'middle' },
  { id: 'pa-pe', d: 'M675,350 C675,412 258,412 258,352', on: [2], prot: 'none', label: 'POST /decide · AccessContext → Decision', lx: 466, ly: 424, anchor: 'middle' },
  { id: 'pe-redis', d: 'M320,323 L393,323', on: [2], prot: 'none', label: 'SISMEMBER ti:*', lx: 324, ly: 288 },
  { id: 'admin-redis', d: 'M142,344 C200,372 330,372 400,350', on: [2], prot: 'none', noProt: true, label: 'redis-cli SADD ti:*', lx: 150, ly: 390 },
  // 3 資料平面
  { id: 'pep-a', d: 'M745,140 C800,140 810,55 856,55', on: [3], prot: 'none', label: 'HTTP + X-ZT-User', lx: 752, ly: 104 },
  { id: 'pep-b', d: 'M745,158 C800,158 815,195 856,195', on: [3], prot: 'none' },
  { id: 'pep-c', d: 'M735,180 C800,240 815,335 856,335', on: [3], prot: 'none' },
  { id: 'a-b', d: 'M960,82 L960,168', on: [3], prot: 'none', blocked: true },
]

const protColor: Record<Prot, string> = {
  ok: 'var(--zt-allow)',
  sig: 'var(--zt-pep)',
  none: 'var(--zt-deny)',
}

const activeNodes: Record<number, string[]> = {
  1: ['browser', 'phone', 'caddy', 'keycloak', 'pep'],
  2: ['pep', 'pa', 'pe', 'redis', 'keycloak', 'admin'],
  3: ['pep', 'res-a', 'res-b', 'res-c'],
}

const captions = [
  '<b>對外只開 Caddy :443</b>；跨在網段邊界上的元件同時接兩個網段，資源只接自己的 enclave',
  '<b>公開入口</b>：Caddy 終止公開 HTTPS，再以 step-ca 憑證連 PEP；不轉送用戶端憑證 → 一律是未受管理裝置',
  '<b>控制平面</b>：每個請求 PEP → PA → PE，PE 查 PIP；PA 寫入授權與稽核，撤銷經 pub/sub 通知 PEP',
  '<b>資料平面</b>：PEP 是唯一能同時連到三個 enclave 的元件；資源之間互相連不到（微分段）',
  '<b>連線保護</b>：外部連線都有 TLS；控制平面內部只靠 internal network 隔離 —— 已知限制',
]

const isProt = computed(() => props.step === 4)
const active = computed(() => activeNodes[props.step] ?? [])

function edgeStyle(e: Edge) {
  if (isProt.value) {
    if (e.blocked)
      return { opacity: 0 }
    if (e.noProt)
      return { opacity: 0.12 }
    return { stroke: protColor[e.prot], opacity: 0.95, strokeDasharray: e.dashed ? '6 5' : 'none', animation: 'none' }
  }
  if (!e.on.includes(props.step)) {
    if (e.blocked)
      return { opacity: 0 }
    return e.dashed ? { strokeDasharray: '6 5' } : undefined
  }
  if (e.blocked)
    return { stroke: 'var(--zt-deny)', opacity: 1 }
  return { stroke: 'var(--zt-fg)', opacity: 1 }
}
</script>

<template>
  <div>
    <svg viewBox="0 0 1000 432" class="zt-svg arch">
      <defs>
        <marker id="ad-arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
          <path d="M0,0 L10,5 L0,10 z" style="fill: var(--zt-line)" />
        </marker>
      </defs>

      <!-- 網段：彼此不重疊；PEP、Keycloak 跨在 untrusted / control-plane 的邊界上 -->
      <rect x="4" y="4" width="152" height="196" rx="12" class="zone" />
      <text x="14" y="194" class="zone-label">客戶端（不受信任）</text>
      <rect x="172" y="4" width="610" height="148" rx="12" class="zone" />
      <text x="772" y="20" text-anchor="end" class="zone-label">untrusted（對外）</text>
      <rect x="172" y="160" width="610" height="268" rx="12" class="zone" style="stroke: var(--zt-ctrl)" />
      <text x="772" y="420" text-anchor="end" class="zone-label" style="fill: var(--zt-ctrl)">control-plane（internal）</text>
      <template v-for="(z, i) in ['a', 'b', 'c']" :key="z">
        <rect x="844" :y="4 + i * 140" width="148" height="128" rx="12" class="zone" />
        <text x="852" :y="124 + i * 140" class="zone-label">enclave-{{ z }}</text>
      </template>

      <!-- 連線 -->
      <path
        v-for="e in edges" :key="e.id"
        :d="e.d"
        class="edge"
        :class="{ on: !isProt && e.on.includes(props.step) && !e.blocked }"
        :style="edgeStyle(e)"
        marker-end="url(#ad-arrow)"
        :marker-start="e.both ? 'url(#ad-arrow)' : undefined"
      />

      <!-- 連線標籤：只顯示目前步驟的 -->
      <template v-for="e in edges" :key="`l-${e.id}`">
        <text
          v-if="e.label"
          :x="e.lx" :y="e.ly"
          :text-anchor="e.anchor ?? 'start'"
          class="edge-label halo zt-fade"
          :class="{ 'zt-hidden': isProt || !e.on.includes(props.step) }"
        ><tspan
          v-for="(line, i) in e.label.split('\n')" :key="i"
          :x="e.lx" :dy="i === 0 ? 0 : 13"
        >{{ line }}</tspan></text>
      </template>

      <!-- 微分段 -->
      <g class="zt-fade" :class="{ 'zt-hidden': props.step !== 3 }">
        <circle cx="960" cy="138" r="10" class="block" />
        <path d="M955,133 L965,143 M965,133 L955,143" class="block-x" />
      </g>

      <!-- 節點 -->
      <SvgNode
        v-for="n in nodes" :key="n.id"
        v-bind="n"
        :active="active.includes(n.id)"
        :dim="props.step > 0 && props.step < 4 && !active.includes(n.id)"
      />
    </svg>

    <Transition name="cap" mode="out-in">
      <div :key="props.step" class="zt-caption text-center text-sm mt-1" v-html="captions[props.step]" />
    </Transition>
    <div class="legend zt-fade" :class="{ 'zt-hidden': !isProt }">
      <span><i style="background: var(--zt-allow)" />TLS + 驗證對端</span>
      <span><i style="background: var(--zt-pep)" />未加密，但 token 有簽章</span>
      <span><i style="background: var(--zt-deny)" />未加密、未驗證（只靠網段隔離）</span>
    </div>
  </div>
</template>

<style scoped>
.arch {
  max-height: 22.5rem;
}

.edge-label {
  font-size: 11px;
  fill: var(--zt-fg) !important;
  font-family: ui-monospace, monospace;
}

.block {
  fill: var(--zt-bg);
  stroke: var(--zt-deny);
  stroke-width: 2;
}

.block-x {
  stroke: var(--zt-deny);
  stroke-width: 2.5;
  stroke-linecap: round;
}

.legend {
  display: flex;
  justify-content: center;
  gap: 1.5rem;
  font-size: 0.75rem;
  margin-top: 0.2rem;
}

.legend i {
  display: inline-block;
  width: 1.4rem;
  height: 3px;
  margin-right: 0.4rem;
  vertical-align: middle;
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
