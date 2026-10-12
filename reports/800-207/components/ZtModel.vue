<script setup lang="ts">
// NIST SP 800-207 Figure 2 的簡化版：資料平面 → 控制平面 → PIP → 本專案對應
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })

const pipsLeft = [
  { label: 'CDM 裝置狀態', impl: 'Redis posture:*（模擬）', y: 20 },
  { label: '威脅情資', impl: 'Redis ti:*（模擬）', y: 72 },
  { label: '活動日誌', impl: 'PEP / PA / PE log', y: 124 },
]
const pipsRight = [
  { label: '存取政策', impl: 'config/policy.json', y: 20 },
  { label: 'ID 管理', impl: 'Keycloak + passkey', y: 72 },
  { label: 'PKI', impl: 'step-ca', y: 124 },
]
</script>

<template>
  <svg viewBox="0 0 640 340" class="zt-svg">
    <defs>
      <marker id="ztm-arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
        <path d="M0,0 L10,5 L0,10 z" style="fill: var(--zt-line)" />
      </marker>
    </defs>

    <!-- 控制平面 / 資料平面分隔 -->
    <g class="zt-fade" :class="{ 'zt-hidden': props.step < 1 }">
      <line x1="0" y1="198" x2="640" y2="198" class="zone" />
      <text x="4" y="190" class="zone-label">控制平面 Control Plane</text>
      <text x="4" y="214" class="zone-label">資料平面 Data Plane</text>
    </g>

    <!-- 資料平面 -->
    <rect x="242" y="236" width="392" height="96" rx="12" class="zone" />
    <text x="626" y="326" text-anchor="end" class="zone-label">implicit trust zone（越小越好）</text>
    <path d="M150,284 L253,284" class="edge" :class="{ on: props.step >= 0 }" marker-end="url(#ztm-arrow)" />
    <text x="200" y="274" text-anchor="middle" class="halo" style="font-size: 11px; fill: var(--zt-deny)">不信任</text>
    <path d="M385,284 L488,284" class="edge" :class="{ on: props.step >= 0 }" marker-end="url(#ztm-arrow)" />
    <SvgNode :x="20" :y="254" :w="130" :h="60" label="主體" sub="使用者 + 裝置" color="var(--zt-res)" />
    <SvgNode :x="255" :y="254" :w="130" :h="60" label="PEP" sub="政策執行點" color="var(--zt-pep)" :active="props.step === 0" />
    <SvgNode :x="490" :y="254" :w="130" :h="60" label="企業資源" sub="Resource" color="var(--zt-res)" />

    <!-- 控制平面 -->
    <g class="zt-fade" :class="{ 'zt-hidden': props.step < 1 }">
      <rect x="205" y="44" width="230" height="110" rx="12" class="zone" />
      <text x="214" y="146" class="zone-label">PDP</text>
      <path d="M320,254 L320,128" class="edge" :class="{ on: props.step >= 1 }" marker-end="url(#ztm-arrow)" marker-start="url(#ztm-arrow)" />
      <text x="328" y="232" class="halo" style="font-size: 11px">每次存取都詢問</text>
      <SvgNode :x="218" :y="68" :w="96" :h="58" label="PE" sub="政策引擎" color="var(--zt-ctrl)" :active="props.step === 1" />
      <SvgNode :x="326" :y="68" :w="96" :h="58" label="PA" sub="政策管理員" color="var(--zt-ctrl)" :active="props.step === 1" />
    </g>

    <!-- PIP -->
    <g class="zt-fade" :class="{ 'zt-hidden': props.step < 2 }">
      <path
        v-for="p in pipsLeft" :key="p.label"
        :d="`M170,${p.y + 22} C190,${p.y + 22} 196,97 216,97`"
        class="edge" :class="{ on: props.step >= 2 }" marker-end="url(#ztm-arrow)"
      />
      <path
        v-for="p in pipsRight" :key="p.label"
        :d="`M470,${p.y + 22} C450,${p.y + 22} 444,97 424,97`"
        class="edge" :class="{ on: props.step >= 2 }" marker-end="url(#ztm-arrow)"
      />
      <SvgNode
        v-for="p in [...pipsLeft.map(p => ({ ...p, x: 10 })), ...pipsRight.map(p => ({ ...p, x: 470 }))]"
        :key="p.label"
        :x="p.x" :y="p.y" :w="160" :h="44"
        :label="p.label"
        :sub="props.step >= 3 ? p.impl : undefined"
        color="var(--zt-pip)"
        :active="props.step === 3"
      />
    </g>
  </svg>
</template>
