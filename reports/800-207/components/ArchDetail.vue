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
  { id: 'caddy', x: 188, y: 34, w: 117, h: 52, label: 'Caddy', sub: ':443 公開 HTTPS', color: 'var(--zt-res)' },
  { id: 'keycloak', x: 335, y: 126, w: 125, h: 54, label: 'Keycloak', sub: ':8080 passkey', tag: 'IdP', color: 'var(--zt-idp)' },
  { id: 'pep', x: 515, y: 126, w: 135, h: 54, label: 'PEP', sub: ':8443 Go proxy', tag: 'PEP', color: 'var(--zt-pep)' },
  { id: 'ca', x: 188, y: 206, w: 117, h: 46, label: 'step-ca', sub: 'PEP 伺服器憑證', tag: 'PKI', color: 'var(--zt-pip)' },
  { id: 'pe', x: 188, y: 296, w: 117, h: 54, label: 'PE', sub: ':8081 /decide', tag: 'PE', color: 'var(--zt-ctrl)' },
  { id: 'redis', x: 340, y: 296, w: 125, h: 54, label: 'Redis', sub: 'PIP · session · 事件', tag: 'PIP', color: 'var(--zt-pip)' },
  { id: 'pa', x: 525, y: 296, w: 115, h: 54, label: 'PA', sub: ':8082 /access', tag: 'PA', color: 'var(--zt-ctrl)' },
  // enclave-media：Lithium Player
  { id: 'lithium', x: 705, y: 30, w: 120, h: 46, label: 'lithium', sub: 'nginx 入口', color: 'var(--zt-res)' },
  { id: 'l-web', x: 860, y: 22, w: 124, h: 40, label: 'lithium-web', sub: 'SvelteKit', color: 'var(--zt-res)' },
  { id: 'l-api', x: 860, y: 82, w: 124, h: 40, label: 'lithium-api', sub: 'FastAPI', color: 'var(--zt-res)' },
  { id: 'l-db', x: 860, y: 142, w: 124, h: 40, label: 'lithium-db', sub: 'Postgres', color: 'var(--zt-res)' },
  // enclave-status：YASS
  { id: 'yass', x: 705, y: 290, w: 120, h: 50, label: 'yass', sub: 'SvelteKit + actix', color: 'var(--zt-res)' },
  { id: 'ag-1', x: 860, y: 244, w: 124, h: 40, label: 'web-01', sub: 'yass-agent', color: 'var(--zt-res)' },
  { id: 'ag-2', x: 860, y: 304, w: 124, h: 40, label: 'db-01', sub: 'yass-agent', color: 'var(--zt-res)' },
  { id: 'ag-3', x: 860, y: 364, w: 124, h: 40, label: 'build-01', sub: 'yass-agent', color: 'var(--zt-res)' },
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
  { id: 'b-caddy', d: 'M142,58 L186,60', on: [1], prot: 'ok', label: 'HTTPS', lx: 146, ly: 50 },
  { id: 'ph-b', d: 'M44,126 L44,84', on: [1], prot: 'ok', both: true, dashed: true, label: 'QR + BLE', lx: 52, ly: 109 },
  { id: 'caddy-pep', d: 'M305,52 C430,30 560,40 580,124', on: [1], prot: 'ok', label: 'HTTPS（step-ca，驗 SAN=pep）· 不轉送用戶端憑證', lx: 318, ly: 28 },
  { id: 'caddy-kc', d: 'M290,86 C320,108 360,100 390,124', on: [1], prot: 'none', label: '/auth/*', lx: 330, ly: 112 },
  // 2 控制平面
  { id: 'pep-kc', d: 'M515,168 L462,168', on: [2], prot: 'sig', label: 'token / JWKS', lx: 488, ly: 196, anchor: 'middle' },
  { id: 'pep-pa', d: 'M582,180 L582,294', on: [2], prot: 'none', label: 'POST /access', lx: 589, ly: 240 },
  { id: 'redis-pep', d: 'M430,294 C450,240 520,220 555,182', on: [2], prot: 'none', label: 'SUBSCRIBE pep:revoke', lx: 330, ly: 236 },
  { id: 'pa-redis', d: 'M525,323 L467,323', on: [2], prot: 'none', label: 'SET session · SADD grants\nPUBLISH · XADD', lx: 470, ly: 364, anchor: 'middle' },
  { id: 'pa-pe', d: 'M582,350 C582,428 247,428 247,352', on: [2], prot: 'none', label: 'POST /decide · AccessContext → Decision', lx: 412, ly: 404, anchor: 'middle' },
  { id: 'pe-redis', d: 'M305,323 L338,323', on: [2], prot: 'none', label: 'SISMEMBER ti:*', lx: 300, ly: 288 },
  { id: 'admin-redis', d: 'M142,344 C200,372 300,372 342,350', on: [2], prot: 'none', noProt: true, label: 'redis-cli SADD ti:*', lx: 150, ly: 420 },
  // 3 資料平面
  { id: 'pep-media', d: 'M650,140 C678,140 680,53 703,53', on: [3], prot: 'none', label: '+ X-ZT-User', lx: 700, ly: 100 },
  { id: 'pep-status', d: 'M650,165 C682,165 680,315 703,315', on: [3], prot: 'none', label: '+ X-ZT-User', lx: 700, ly: 274 },
  { id: 'l-web', d: 'M825,46 L858,42', on: [3], prot: 'none' },
  { id: 'l-api', d: 'M825,60 L858,100', on: [3], prot: 'none' },
  { id: 'l-db', d: 'M922,122 L922,140', on: [3], prot: 'none' },
  { id: 'ag-1', d: 'M858,264 L827,302', on: [3], prot: 'sig' },
  { id: 'ag-2', d: 'M858,324 L827,316', on: [3], prot: 'sig' },
  { id: 'ag-3', d: 'M858,384 L827,330', on: [3], prot: 'sig', label: 'agent：Bearer 主機 token', lx: 700, ly: 366 },
  { id: 'l-yass', d: 'M810,78 L810,288', on: [3], prot: 'none', blocked: true },
]

const protColor: Record<Prot, string> = {
  ok: 'var(--zt-allow)',
  sig: 'var(--zt-pep)',
  none: 'var(--zt-deny)',
}

const activeNodes: Record<number, string[]> = {
  1: ['browser', 'phone', 'caddy', 'keycloak', 'pep'],
  2: ['pep', 'pa', 'pe', 'redis', 'keycloak', 'admin'],
  3: ['pep', 'lithium', 'l-web', 'l-api', 'l-db', 'yass', 'ag-1', 'ag-2', 'ag-3'],
}

const captions = [
  '<b>對外只開 Caddy :443</b>；跨在網段邊界上的元件同時接兩個網段，每個應用只接自己的 enclave',
  '<b>公開入口</b>：Caddy 終止公開 HTTPS，再以 step-ca 憑證連 PEP；不轉送用戶端憑證 → 一律是未受管理裝置',
  '<b>控制平面</b>：每個請求 PEP → PA → PE，PE 查 PIP；PA 寫入授權與稽核，撤銷經 pub/sub 通知 PEP',
  '<b>資料平面</b>：只有 PEP 同時接上兩個 enclave；應用信任 PEP 寫入的 <code>X-ZT-User</code>。lithium 連得到自己的後端，連不到 yass',
  '<b>連線保護</b>：外部連線都有 TLS；控制平面與 enclave 內部只靠 internal network 隔離 —— 已知限制',
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
    <svg viewBox="0 0 1000 440" class="zt-svg arch">
      <defs>
        <marker id="ad-arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
          <path d="M0,0 L10,5 L0,10 z" style="fill: var(--zt-line)" />
        </marker>
      </defs>

      <!-- 網段：彼此不重疊；PEP、Keycloak 跨在 untrusted / control-plane 的邊界上 -->
      <rect x="4" y="4" width="152" height="196" rx="12" class="zone" />
      <text x="14" y="194" class="zone-label">客戶端（不受信任）</text>
      <rect x="170" y="4" width="500" height="148" rx="12" class="zone" />
      <text x="180" y="20" class="zone-label">untrusted（對外）</text>
      <rect x="170" y="160" width="500" height="276" rx="12" class="zone" style="stroke: var(--zt-ctrl)" />
      <text x="660" y="428" text-anchor="end" class="zone-label" style="fill: var(--zt-ctrl)">control-plane（internal）</text>
      <rect x="690" y="4" width="306" height="208" rx="12" class="zone" />
      <text x="700" y="20" class="zone-label">enclave-media</text>
      <text x="700" y="204" class="zone-label">Lithium Player</text>
      <rect x="690" y="224" width="306" height="212" rx="12" class="zone" />
      <text x="700" y="240" class="zone-label">enclave-status</text>
      <text x="700" y="428" class="zone-label">YASS 伺服器狀態</text>

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

      <!-- 微分段：lithium 連不到 yass -->
      <g class="zt-fade" :class="{ 'zt-hidden': props.step !== 3 }">
        <circle cx="810" cy="218" r="10" class="block" />
        <path d="M805,213 L815,223 M815,213 L805,223" class="block-x" />
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
      <span><i style="background: var(--zt-pep)" />未加密，但靠 token 驗證</span>
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
