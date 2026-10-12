<script setup lang="ts">
import { computed } from 'vue'

// 本專案的實際部署：每一次點擊點亮一條路徑
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })

const nodes = [
  { id: 'browser', x: 20, y: 170, w: 140, h: 60, label: '借用電腦', sub: '只有瀏覽器', color: 'var(--zt-res)' },
  { id: 'phone', x: 20, y: 330, w: 140, h: 60, label: '手機', sub: 'passkey 驗證器', color: 'var(--zt-idp)' },
  { id: 'caddy', x: 215, y: 170, w: 140, h: 60, label: 'Caddy', sub: '公開 HTTPS 入口', color: 'var(--zt-res)' },
  { id: 'keycloak', x: 410, y: 40, w: 160, h: 60, label: 'Keycloak', sub: 'WebAuthn Passwordless', tag: 'IdP', color: 'var(--zt-idp)' },
  { id: 'pep', x: 410, y: 170, w: 160, h: 60, label: 'PEP', sub: 'Go reverse proxy', tag: 'PEP', color: 'var(--zt-pep)' },
  { id: 'pa', x: 385, y: 330, w: 120, h: 60, label: 'PA', sub: '記錄授權 / 撤銷', tag: 'PA', color: 'var(--zt-ctrl)' },
  { id: 'pe', x: 545, y: 330, w: 120, h: 60, label: 'PE', sub: '條件式政策判斷', tag: 'PE', color: 'var(--zt-ctrl)' },
  { id: 'redis', x: 697, y: 330, w: 112, h: 60, label: 'Redis', sub: 'CDM / TI（模擬）', tag: 'PIP', color: 'var(--zt-pip)' },
  { id: 'res-a', x: 800, y: 52, w: 135, h: 56, label: 'res-a', sub: '一般資源 A', color: 'var(--zt-res)' },
  { id: 'res-b', x: 800, y: 184, w: 135, h: 56, label: 'res-b', sub: '敏感資源 B', color: 'var(--zt-res)' },
]

const edges = [
  { id: 'b-c', d: 'M160,200 L213,200', on: [1, 2, 3] },
  { id: 'c-kc', d: 'M355,185 C385,185 382,70 408,70', on: [1] },
  { id: 'ph-b', d: 'M90,328 L90,232', on: [1], color: 'var(--zt-idp)', both: true },
  { id: 'c-pep', d: 'M355,200 L408,200', on: [2, 3] },
  { id: 'pep-pa', d: 'M445,230 L445,328', on: [2], both: true },
  { id: 'pa-pe', d: 'M507,360 L543,360', on: [2], both: true },
  { id: 'pe-redis', d: 'M667,360 L695,360', on: [2] },
  { id: 'pep-a', d: 'M570,188 C690,188 680,80 798,80', on: [3], color: 'var(--zt-allow)' },
  { id: 'pep-b', d: 'M570,212 L798,212', on: [3], color: 'var(--zt-deny)', blocked: true },
  { id: 'a-b', d: 'M922,108 L922,182', on: [4], color: 'var(--zt-deny)', blocked: true },
]

const activeNodes: Record<number, string[]> = {
  1: ['browser', 'phone', 'caddy', 'keycloak'],
  2: ['browser', 'caddy', 'pep', 'pa', 'pe', 'redis'],
  3: ['pep', 'res-a', 'res-b'],
  4: ['res-a', 'res-b'],
}

const captions = [
  '<b>只有 Caddy 對外</b>：PEP 是通往資源的唯一入口，其餘服務都在 internal network',
  '<b>登入</b>：瀏覽器經 Caddy 到 Keycloak，手機用 passkey 完成驗證',
  '<b>每一個請求</b>：PEP → PA → PE，PE 查 Redis 的裝置狀態與威脅情資',
  '<b>放行才轉發</b>：A 允許未受管理裝置；B 回 403「需要受管理裝置」',
  '<b>微分段</b>：res-a、res-b 在不同 internal network，就算 res-a 被攻陷也連不到 B',
]

const active = computed(() => activeNodes[props.step] ?? [])
const isDim = (id: string) => props.step > 0 && !active.value.includes(id)
</script>

<template>
  <div>
    <svg viewBox="0 0 960 420" class="zt-svg">
      <defs>
        <marker id="arch-arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
          <path d="M0,0 L10,5 L0,10 z" style="fill: var(--zt-line)" />
        </marker>
      </defs>

      <!-- 網路區域 -->
      <rect x="6" y="150" width="168" height="258" rx="12" class="zone" />
      <text x="16" y="402" class="zone-label">使用者端（不受信任）</text>
      <rect x="375" y="306" width="442" height="104" rx="12" class="zone" />
      <text x="385" y="402" class="zone-label" style="fill: var(--zt-ctrl)">control-plane（internal）</text>
      <rect x="787" y="22" width="161" height="102" rx="12" class="zone" />
      <text x="797" y="118" class="zone-label">enclave-a（internal）</text>
      <rect x="787" y="154" width="161" height="102" rx="12" class="zone" />
      <text x="797" y="250" class="zone-label">enclave-b（internal）</text>

      <!-- 連線 -->
      <path
        v-for="e in edges" :key="e.id"
        :d="e.d"
        class="edge"
        :class="{ on: e.on.includes(props.step) && !e.blocked }"
        :style="e.on.includes(props.step) ? { stroke: e.color ?? 'var(--zt-fg)', opacity: 1 } : undefined"
        marker-end="url(#arch-arrow)"
        :marker-start="e.both ? 'url(#arch-arrow)' : undefined"
      />

      <text x="186" y="190" text-anchor="middle" class="edge-label">443</text>
      <text x="372" y="122" class="edge-label halo zt-fade" :class="{ 'zt-hidden': props.step !== 1 }">/auth/*</text>
      <text x="100" y="285" class="edge-label halo zt-fade" :class="{ 'zt-hidden': props.step !== 1 }" style="fill: var(--zt-idp)">QR + 藍牙</text>
      <text x="455" y="284" class="edge-label halo zt-fade" :class="{ 'zt-hidden': props.step !== 2 }">AccessContext</text>
      <text x="372" y="190" text-anchor="middle" class="edge-label halo zt-fade" :class="{ 'zt-hidden': props.step !== 2 }">TLS</text>

      <!-- 被擋下的連線 -->
      <g class="zt-fade" :class="{ 'zt-hidden': props.step !== 3 }">
        <circle cx="690" cy="212" r="12" class="block" />
        <path d="M684,206 L696,218 M696,206 L684,218" class="block-x" />
        <text x="690" y="244" text-anchor="middle" class="edge-label halo" style="fill: var(--zt-deny)">403 需要受管理裝置</text>
        <text x="680" y="128" text-anchor="middle" class="edge-label halo" style="fill: var(--zt-allow)">200 允許</text>
      </g>
      <g class="zt-fade" :class="{ 'zt-hidden': props.step !== 4 }">
        <circle cx="922" cy="145" r="12" class="block" />
        <path d="M916,139 L928,151 M928,139 L916,151" class="block-x" />
      </g>

      <!-- 節點 -->
      <SvgNode
        v-for="n in nodes" :key="n.id"
        v-bind="n"
        :active="active.includes(n.id)"
        :dim="isDim(n.id)"
      />
    </svg>

    <Transition name="cap" mode="out-in">
      <div :key="props.step" class="zt-caption text-center mt-1" v-html="captions[props.step]" />
    </Transition>
  </div>
</template>

<style scoped>
.edge-label {
  font-size: 11px;
  fill: var(--zt-muted) !important;
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
