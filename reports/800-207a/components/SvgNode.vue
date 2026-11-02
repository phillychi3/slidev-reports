<script setup lang="ts">
// SVG 圖表裡的一個方塊：主標題、副標題、右上角的 800-207 角色標籤
defineProps<{
  x: number
  y: number
  w: number
  h: number
  label: string
  sub?: string
  tag?: string
  color: string
  active?: boolean
  dim?: boolean
}>()
</script>

<template>
  <g class="svg-node" :class="{ active, dim }" :style="{ '--c': color }">
    <rect :x="x" :y="y" :width="w" :height="h" rx="10" class="box" />
    <text
      :x="x + w / 2"
      :y="sub ? y + h / 2 - 3 : y + h / 2 + 5"
      text-anchor="middle"
      class="label"
    >{{ label }}</text>
    <text
      v-if="sub"
      :x="x + w / 2"
      :y="y + h / 2 + 15"
      text-anchor="middle"
      class="sub"
    >{{ sub }}</text>
    <g v-if="tag">
      <rect :x="x + w - tag.length * 7.2 - 10" :y="y - 9" :width="tag.length * 7.2 + 14" height="18" rx="9" class="tag-bg" />
      <text :x="x + w - tag.length * 3.6 - 3" :y="y + 4" text-anchor="middle" class="tag">{{ tag }}</text>
    </g>
  </g>
</template>

<style scoped>
.box {
  /* 不透明底色：跨在網段邊界上的節點不會被網段虛線穿過 */
  fill: color-mix(in srgb, var(--c) 14%, var(--zt-bg));
  stroke: var(--c);
  stroke-width: 1.5;
  transition: fill 0.35s, stroke-width 0.35s;
}

.label {
  font-size: 15px;
  font-weight: 700;
}

.sub {
  font-size: 11px;
  fill: var(--zt-muted) !important;
}

.tag-bg {
  fill: var(--c);
}

.tag {
  font-size: 11px;
  font-weight: 700;
  fill: var(--zt-bg) !important;
  font-family: ui-monospace, monospace;
}

.svg-node {
  transition: opacity 0.4s;
}

.active .box {
  fill: color-mix(in srgb, var(--c) 30%, var(--zt-bg));
  stroke-width: 3;
}

.dim {
  opacity: 0.35;
}
</style>
