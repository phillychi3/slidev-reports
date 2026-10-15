<script setup lang="ts">
import { computed } from 'vue'

// 通用循序圖：step 0 顯示第 1 組訊息，之後每次點擊多一組。
// 訊息種類：from→to 箭頭、只有 from 的自我訊息、note: true 的註記方塊（橫跨 from..to）。
export interface SeqActor { id: string, label: string, color: string }
export interface SeqMsg {
  g: number
  from: string
  to?: string
  text: string
  dashed?: boolean
  note?: boolean
  tone?: 'allow' | 'deny'
}

const props = withDefaults(defineProps<{
  actors: SeqActor[]
  msgs: SeqMsg[]
  captions?: string[]
  step?: number
  height?: number
}>(), { step: 0, captions: () => [], height: 430 })

const W = 960
const TOP = 58
const xs = computed(() => {
  const n = props.actors.length
  const left = 85
  const span = W - left * 2
  return Object.fromEntries(props.actors.map((a, i) => [a.id, left + (n === 1 ? 0 : (i * span) / (n - 1))]))
})
const gap = computed(() => Math.min(30, (props.height - TOP - 10) / props.msgs.length))
const shown = computed(() => props.step + 1)

// 粗估文字寬度（11.5px 字級）：全形字約 11.5px，半形約 6.4px
function textWidth(t: string) {
  let w = 0
  for (const ch of t)
    w += ch.charCodeAt(0) > 0x2E80 ? 11.5 : 6.4
  return w
}

const rows = computed(() => props.msgs.map((m, i) => {
  const y = TOP + gap.value * (i + 0.5)
  const x1 = xs.value[m.from]
  if (m.note) {
    const x2 = xs.value[m.to ?? m.from]
    const mid = (x1 + x2) / 2
    const half = Math.max(Math.abs(x2 - x1) / 2 + 70, textWidth(m.text) / 2 + 14)
    const lo = Math.max(6, mid - half)
    const hi = Math.min(W - 6, mid + half)
    return { ...m, kind: 'note', y, rx: lo, rw: hi - lo, rh: gap.value - 6, tx: (lo + hi) / 2, ty: y + 4, anchor: 'middle' }
  }
  if (!m.to) {
    const dir = x1 > W - 200 ? -1 : 1
    const h = Math.min(14, gap.value * 0.5)
    return {
      ...m, kind: 'self', y,
      d: `M${x1},${y - h / 2} h${dir * 24} v${h} h${-dir * 22}`,
      tx: x1 + dir * 32, ty: y + 4, anchor: dir > 0 ? 'start' : 'end',
    }
  }
  const x2 = xs.value[m.to]
  const dir = x2 > x1 ? 1 : -1
  return {
    ...m, kind: 'arrow', y: y + 6,
    d: `M${x1 + dir * 3},${y + 6} L${x2 - dir * 4},${y + 6}`,
    tx: (x1 + x2) / 2, ty: y + 0.5, anchor: 'middle',
  }
}))

const uid = Math.random().toString(36).slice(2, 8)
</script>

<template>
  <div>
    <svg :viewBox="`0 0 ${W} ${props.height}`" class="zt-svg">
      <defs>
        <marker :id="`seq-${uid}`" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
          <path d="M0,0 L10,5 L0,10 z" style="fill: currentColor" />
        </marker>
      </defs>

      <g v-for="a in props.actors" :key="a.id">
        <line :x1="xs[a.id]" y1="40" :x2="xs[a.id]" :y2="props.height - 4" class="lifeline" />
        <rect :x="xs[a.id] - 72" y="6" width="144" height="34" rx="8" class="actor" :style="{ '--c': a.color }" />
        <text :x="xs[a.id]" y="28" text-anchor="middle" class="actor-label">{{ a.label }}</text>
      </g>

      <g
        v-for="(r, i) in rows" :key="i"
        class="msg"
        :class="[r.kind, r.tone, { shown: r.g <= shown, current: r.g === shown, dashed: r.dashed }]"
      >
        <template v-if="r.kind === 'note'">
          <rect :x="r.rx" :y="r.y - r.rh / 2" :width="r.rw" :height="r.rh" rx="6" class="note-box" />
        </template>
        <path
          v-else
          :d="r.d"
          :pathLength="r.dashed ? undefined : 1"
          class="msg-line"
          :marker-end="`url(#seq-${uid})`"
        />
        <text :x="r.tx" :y="r.ty" :text-anchor="r.anchor" class="msg-text" :class="{ halo: r.kind !== 'note' }">{{ r.text }}</text>
      </g>
    </svg>

    <Transition v-if="props.captions.length" name="cap" mode="out-in">
      <div :key="shown" class="zt-caption text-center" v-html="props.captions[shown - 1]" />
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
  transform: translateY(5px);
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

.msg.current.allow {
  color: var(--zt-allow);
}

.msg.current.deny {
  color: var(--zt-deny);
}

.msg-line {
  fill: none;
  stroke: currentColor;
  stroke-width: 1.7;
  stroke-dasharray: 1;
  stroke-dashoffset: 1;
  transition: stroke-dashoffset 0.55s ease 0.1s;
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
  font-size: 12px;
  fill: var(--zt-muted) !important;
}

.msg.current .msg-text {
  fill: var(--zt-fg) !important;
  font-weight: 600;
}

.note-box {
  fill: color-mix(in srgb, var(--zt-pep) 10%, var(--zt-bg));
  stroke: color-mix(in srgb, var(--zt-pep) 60%, transparent);
  stroke-width: 1;
}

.msg.note .msg-text {
  font-size: 11.5px;
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
