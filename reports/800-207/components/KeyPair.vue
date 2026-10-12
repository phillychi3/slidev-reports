<script setup lang="ts">
// passkey 的 challenge-response；step >= 1 時顯示釣魚網站拿不到簽章
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })
</script>

<template>
  <svg viewBox="0 0 440 300" class="zt-svg">
    <defs>
      <marker id="kp-arrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
        <path d="M0,0 L10,5 L0,10 z" style="fill: var(--zt-line)" />
      </marker>
    </defs>

    <rect x="10" y="30" width="140" height="96" rx="10" class="box" style="--c: var(--zt-idp)" />
    <text x="80" y="58" text-anchor="middle" class="title">手機</text>
    <text x="80" y="88" text-anchor="middle" class="small">🔑 私鑰在安全晶片</text>
    <text x="80" y="108" text-anchor="middle" class="small">本地生物辨識</text>

    <rect x="290" y="30" width="140" height="96" rx="10" class="box" style="--c: var(--zt-idp)" />
    <text x="360" y="58" text-anchor="middle" class="title">zt.cloudowo.com</text>
    <text x="360" y="88" text-anchor="middle" class="small">只存公鑰</text>
    <text x="360" y="108" text-anchor="middle" class="small">沒有可外洩的秘密</text>

    <path d="M288,58 L152,58" class="edge on" marker-end="url(#kp-arrow)" />
    <path d="M152,100 L288,100" class="edge on" marker-end="url(#kp-arrow)" />
    <g class="packet left">
      <rect x="-38" y="-11" width="76" height="22" rx="11" class="pk challenge" />
      <text y="4" text-anchor="middle" class="pk-text">challenge</text>
    </g>
    <g class="packet right">
      <rect x="-30" y="-11" width="60" height="22" rx="11" class="pk sig" />
      <text y="4" text-anchor="middle" class="pk-text">簽章</text>
    </g>

    <!-- 釣魚網站 -->
    <g class="zt-fade" :class="{ 'zt-hidden': props.step < 1 }">
      <rect x="290" y="190" width="140" height="70" rx="10" class="box" style="--c: var(--zt-deny)" />
      <text x="360" y="220" text-anchor="middle" class="title">zt-cloudow0.com</text>
      <text x="360" y="242" text-anchor="middle" class="small">釣魚網站</text>
      <path d="M288,225 C220,225 120,225 80,128" class="edge" style="opacity: 1; stroke: var(--zt-deny); stroke-dasharray: 5 5" marker-end="url(#kp-arrow)" />
      <circle cx="170" cy="214" r="12" class="block" />
      <path d="M164,208 L176,220 M176,208 L164,220" class="block-x" />
      <text x="170" y="250" text-anchor="middle" class="small halo warn">RP ID 不符 → 不會簽章</text>
    </g>
  </svg>
</template>

<style scoped>
.box {
  fill: color-mix(in srgb, var(--c) 14%, transparent);
  stroke: var(--c);
  stroke-width: 1.5;
}

.title {
  font-size: 14px;
  font-weight: 700;
}

.small {
  font-size: 12px;
  fill: var(--zt-muted) !important;
}

.warn {
  fill: var(--zt-deny) !important;
}

.packet {
  animation-duration: 3.2s;
  animation-iteration-count: infinite;
  animation-timing-function: ease-in-out;
  opacity: 0;
}

.packet.left {
  animation-name: to-left;
}

.packet.right {
  animation-name: to-right;
}

@keyframes to-left {
  0% { transform: translate(255px, 58px); opacity: 0; }
  8% { opacity: 1; }
  42% { transform: translate(185px, 58px); opacity: 1; }
  50%, 100% { transform: translate(185px, 58px); opacity: 0; }
}

@keyframes to-right {
  0%, 50% { transform: translate(185px, 100px); opacity: 0; }
  58% { opacity: 1; }
  92% { transform: translate(255px, 100px); opacity: 1; }
  100% { transform: translate(255px, 100px); opacity: 0; }
}

.pk.challenge {
  fill: var(--zt-ctrl);
}

.pk.sig {
  fill: var(--zt-idp);
}

.pk-text {
  font-size: 11px;
  font-weight: 700;
  font-family: ui-monospace, monospace;
  fill: var(--zt-bg) !important;
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
</style>
