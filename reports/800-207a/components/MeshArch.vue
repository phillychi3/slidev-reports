<script setup lang="ts">
import { computed } from 'vue'

// GKE + Cloud Service Mesh 部署圖：每個 namespace 一個信任區。
// step 1–3 依類別點亮連線並標出傳遞內容，step 4 依連線保護等級上色。
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })

type Prot = 'mtls' | 'claim' | 'ext'

const nodes = [
  { id: 'browser', x: 16, y: 16, w: 126, h: 46, label: '借用電腦', sub: '瀏覽器 · 無裝置憑證', color: 'var(--zt-res)' },
  { id: 'phone', x: 16, y: 100, w: 126, h: 46, label: '手機', sub: 'passkey', color: 'var(--zt-idp)' },
  { id: 'laptop', x: 16, y: 172, w: 126, h: 46, label: '受管理電腦', sub: '裝置憑證 laptop1', color: 'var(--zt-res)' },
  { id: 'meshca', x: 16, y: 300, w: 126, h: 54, label: 'Mesh CA', sub: '代管控制平面', tag: 'PKI', color: 'var(--zt-pip)' },
  { id: 'gw', x: 178, y: 96, w: 114, h: 60, label: 'ingress gw', sub: 'L4 LB → Envoy', tag: 'PEP', color: 'var(--zt-pep)' },
  { id: 'keycloak', x: 332, y: 28, w: 120, h: 50, label: 'Keycloak', sub: 'passkey · OIDC', tag: 'IdP', sidecar: true, color: 'var(--zt-idp)' },
  { id: 'pep', x: 520, y: 28, w: 148, h: 50, label: 'PEP', sub: 'pep · pep-managed', tag: 'PEP', sidecar: true, color: 'var(--zt-pep)' },
  { id: 'pa', x: 534, y: 160, w: 120, h: 50, label: 'PA', sub: '/access', tag: 'PA', sidecar: true, color: 'var(--zt-ctrl)' },
  { id: 'pe', x: 332, y: 300, w: 120, h: 50, label: 'PE', sub: '/decide', tag: 'PE', sidecar: true, color: 'var(--zt-ctrl)' },
  { id: 'redis', x: 534, y: 300, w: 120, h: 50, label: 'Redis', sub: 'session · 事件', tag: 'PIP', sidecar: true, color: 'var(--zt-pip)' },
  // zt-media：Lithium Player
  { id: 'lithium', x: 715, y: 30, w: 120, h: 46, label: 'lithium', sub: 'nginx 入口', sidecar: true, color: 'var(--zt-res)' },
  { id: 'l-web', x: 860, y: 22, w: 124, h: 40, label: 'lithium-web', sub: 'SvelteKit', sidecar: true, color: 'var(--zt-res)' },
  { id: 'l-api', x: 860, y: 82, w: 124, h: 40, label: 'lithium-api', sub: 'FastAPI', sidecar: true, color: 'var(--zt-res)' },
  { id: 'l-db', x: 860, y: 142, w: 124, h: 40, label: 'lithium-db', sub: 'Postgres', sidecar: true, color: 'var(--zt-res)' },
  // zt-status：YASS
  { id: 'yass', x: 715, y: 290, w: 120, h: 50, label: 'yass', sub: 'SvelteKit + actix', sidecar: true, color: 'var(--zt-res)' },
  { id: 'ag-1', x: 860, y: 244, w: 124, h: 40, label: 'web-01', sub: 'yass-agent', sidecar: true, color: 'var(--zt-res)' },
  { id: 'ag-2', x: 860, y: 304, w: 124, h: 40, label: 'db-01', sub: 'yass-agent', sidecar: true, color: 'var(--zt-res)' },
  { id: 'ag-3', x: 860, y: 364, w: 124, h: 40, label: 'build-01', sub: 'yass-agent', sidecar: true, color: 'var(--zt-res)' },
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
  noProt?: boolean // 不是請求路徑（設定與憑證下發），保護等級那一步不顯示
}

const edges: Edge[] = [
  { id: 'ca', d: 'M142,327 L318,327', on: [0], prot: 'mtls', dashed: true, noProt: true, label: 'xDS 設定 · SPIFFE 憑證', lx: 150, ly: 318 },
  // 1 入口
  { id: 'b-gw', d: 'M142,39 C164,39 166,100 176,108', on: [1], prot: 'ext', label: 'HTTPS', lx: 146, ly: 30 },
  { id: 'ph-b', d: 'M44,98 L44,64', on: [1], prot: 'ext', both: true, dashed: true, label: 'QR + BLE', lx: 52, ly: 85 },
  { id: 'l-gw', d: 'M142,195 C164,195 166,150 176,144', on: [1], prot: 'ext', label: 'mTLS 裝置憑證', lx: 146, ly: 232 },
  { id: 'gw-pep', d: 'M292,106 C326,8 500,4 560,26', on: [1], prot: 'mtls', label: 'XFCC（上一跳 = gateway）', lx: 404, ly: 14, anchor: 'middle' },
  { id: 'gw-kc', d: 'M292,116 C312,100 314,60 330,56', on: [1], prot: 'mtls', label: '/auth/*', lx: 298, ly: 86 },
  // 2 控制平面
  { id: 'pep-kc', d: 'M520,53 L454,53', on: [2], prot: 'mtls', label: 'token / JWKS', lx: 487, ly: 94, anchor: 'middle' },
  { id: 'pep-pa', d: 'M594,78 L594,158', on: [2], prot: 'mtls', label: 'POST /access', lx: 588, ly: 112, anchor: 'end' },
  { id: 'redis-pep', d: 'M654,318 C686,300 686,90 668,64', on: [2], prot: 'mtls', label: 'SUBSCRIBE pep:revoke', lx: 680, ly: 140, anchor: 'end' },
  { id: 'pa-redis', d: 'M614,210 L614,298', on: [2], prot: 'mtls', label: 'grants\nXADD', lx: 620, ly: 250 },
  { id: 'pa-pe', d: 'M548,210 C530,262 470,262 442,298', on: [2], prot: 'mtls', label: 'POST /decide', lx: 470, ly: 240, anchor: 'middle' },
  { id: 'pe-redis', d: 'M452,325 L532,325', on: [2], prot: 'mtls', label: 'SISMEMBER ti:*', lx: 492, ly: 368, anchor: 'middle' },
  // 3 資料平面
  { id: 'pep-media', d: 'M668,44 C690,44 690,53 713,53', on: [3], prot: 'claim', label: '+ X-ZT-Assertion\naud=media', lx: 712, ly: 96 },
  { id: 'pep-status', d: 'M668,68 C694,68 690,315 713,315', on: [3], prot: 'claim', label: '+ X-ZT-Assertion\naud=status\n需 admin', lx: 712, ly: 258 },
  { id: 'l-web', d: 'M835,46 L858,42', on: [3], prot: 'claim' },
  { id: 'l-api', d: 'M835,62 L858,100', on: [3], prot: 'claim' },
  { id: 'l-db', d: 'M922,122 L922,140', on: [3], prot: 'mtls' },
  { id: 'ag-1', d: 'M858,264 L837,302', on: [3], prot: 'mtls' },
  { id: 'ag-2', d: 'M858,324 L837,316', on: [3], prot: 'mtls' },
  { id: 'ag-3', d: 'M858,384 L837,330', on: [3], prot: 'mtls', label: 'Bearer 主機 token', lx: 722, ly: 372 },
  { id: 'l-yass', d: 'M828,78 L828,288', on: [3], prot: 'mtls', blocked: true },
]

const protColor: Record<Prot, string> = {
  mtls: 'var(--zt-allow)',
  claim: 'var(--zt-ctrl)',
  ext: 'var(--zt-pep)',
}

const activeNodes: Record<number, string[]> = {
  1: ['browser', 'phone', 'laptop', 'gw', 'pep', 'keycloak'],
  2: ['pep', 'pa', 'pe', 'redis', 'keycloak'],
  3: ['pep', 'lithium', 'l-web', 'l-api', 'l-db', 'yass', 'ag-1', 'ag-2', 'ag-3'],
}

const captions = [
  '<b>四個 namespace 各是一個信任區</b>；每個 workload 旁注入 Envoy sidecar，以自己的 ServiceAccount ＝ SPIFFE 身分做 mTLS，憑證由代管 Mesh CA 簽發、自動輪替',
  '<b>入口</b>：只有 ingress gateway 對外。公開網域不收用戶端憑證；裝置網域走 TLS MUTUAL，裝置資訊經 XFCC 交給 PEP',
  '<b>控制平面</b>：PEP → PA → PE → Redis 每一跳都是 sidecar 之間的 mTLS，再由 AuthorizationPolicy 檢查「誰 → 哪個方法／路徑」',
  '<b>資料平面</b>：PEP 對每個請求簽發 <code>X-ZT-Assertion</code>；資源前的 sidecar 驗證 SPIFFE 身分＋聲明。lithium 連不到 yass',
  '<b>連線保護</b>：除了申請憑證時的 ACME 驗證，每一條服務間連線都是雙向驗證的 mTLS',
]

const isProt = computed(() => props.step === 4)
const active = computed(() => activeNodes[props.step] ?? [])

function edgeStyle(e: Edge) {
  if (isProt.value) {
    if (e.blocked)
      return { opacity: 0 }
    if (e.noProt)
      return { opacity: 0.12, strokeDasharray: '6 5' }
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
        <marker id="ma-arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
          <path d="M0,0 L10,5 L0,10 z" style="fill: var(--zt-line)" />
        </marker>
      </defs>

      <!-- 信任區：每個 namespace 預設進出皆拒（網路層）＋ 預設 allow-nothing（身分層） -->
      <rect x="4" y="4" width="152" height="236" rx="12" class="zone" />
      <text x="14" y="252" class="zone-label">客戶端（不受信任）</text>
      <rect x="4" y="286" width="152" height="80" rx="12" class="zone" />
      <text x="14" y="380" class="zone-label">Google 代管</text>
      <rect x="166" y="66" width="138" height="112" rx="12" class="zone ns" />
      <text x="176" y="170" class="zone-label ns-label">zt-gateway</text>
      <rect x="318" y="4" width="370" height="432" rx="12" class="zone ns" />
      <text x="678" y="428" text-anchor="end" class="zone-label ns-label">zt-system（控制平面）</text>
      <rect x="700" y="4" width="296" height="208" rx="12" class="zone ns" />
      <text x="986" y="204" text-anchor="end" class="zone-label ns-label">zt-media · Lithium Player</text>
      <rect x="700" y="224" width="296" height="212" rx="12" class="zone ns" />
      <text x="710" y="240" class="zone-label ns-label">zt-status · YASS</text>

      <!-- 圖例：節點左側的綠條 = 注入的 Envoy sidecar；ingress gw 本身就是 Envoy，不另畫 -->
      <rect x="170" y="254" width="24" height="26" rx="5" class="key-box" />
      <rect x="174" y="258" width="6" height="18" rx="3" class="key-sidecar" />
      <text x="200" y="265" class="key-text">Envoy sidecar</text>
      <text x="200" y="279" class="key-text muted">mTLS · 授權執行點</text>

      <!-- 連線 -->
      <path
        v-for="e in edges" :key="e.id"
        :d="e.d"
        class="edge"
        :class="{ on: !isProt && e.on.includes(props.step) && !e.blocked }"
        :style="edgeStyle(e)"
        marker-end="url(#ma-arrow)"
        :marker-start="e.both ? 'url(#ma-arrow)' : undefined"
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

      <!-- 微分段：lithium 連不到 yass（身分層出站限制＋網路層） -->
      <g class="zt-fade" :class="{ 'zt-hidden': props.step !== 3 }">
        <circle cx="828" cy="218" r="10" class="block" />
        <path d="M823,213 L833,223 M833,213 L823,223" class="block-x" />
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
      <span><i style="background: var(--zt-pep)" />外部：公開 HTTPS／裝置 mTLS／FIDO</span>
      <span><i style="background: var(--zt-allow)" />mTLS＋SPIFFE＋服務授權</span>
      <span><i style="background: var(--zt-ctrl)" />再加上使用者聲明（X-ZT-Assertion）</span>
    </div>
  </div>
</template>

<style scoped>
.arch {
  max-height: 22.5rem;
}

.ns {
  stroke: var(--zt-ctrl);
}

.ns-label {
  fill: var(--zt-ctrl) !important;
}

.edge-label {
  font-size: 11px;
  fill: var(--zt-fg) !important;
  font-family: ui-monospace, monospace;
}

.key-box {
  fill: none;
  stroke: var(--zt-line);
  stroke-width: 1.2;
}

.key-sidecar {
  fill: var(--zt-allow);
}

.key-text {
  font-size: 11px;
  font-family: ui-monospace, monospace;
}

.key-text.muted {
  fill: var(--zt-muted) !important;
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
