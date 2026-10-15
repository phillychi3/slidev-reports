<script setup lang="ts">
import { computed } from 'vue'

// 帳號被列入風險名單後的撤銷流程；log 格式對應 PE / PA / PEP 的 log.Printf
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })

const stations = [
  { title: '風險事件', body: '管理者寫入威脅情資', code: 'SADD ti:bad_users', color: 'var(--zt-pip)' },
  { title: 'Redis 通知', body: 'keyspace notification；PA 記一筆 pip 事件', code: 'keyspace: ti:*', color: 'var(--zt-pip)' },
  { title: 'PA 重新評估', body: '已授權的資源全部重問 PE（每 30 秒也會跑一次）', code: 'reevaluateAll()', color: 'var(--zt-ctrl)' },
  { title: '撤銷', body: 'SREM grants、通知 PEP、記 revoke 事件', code: 'PUBLISH pep:revoke', color: 'var(--zt-ctrl)' },
  { title: '下一個請求', body: '重新整理 /a/，PEP 照常再問', code: 'HTTP 403', color: 'var(--zt-deny)' },
]

const reason = '使用者帳號在威脅情資名單中'
const sid = '9c41d07be2a3'
const logs = [
  { s: 0, t: 'cmd', text: '$ redis-cli SADD ti:bad_users alice' },
  { s: 0, t: 'out', text: '(integer) 1' },
  { s: 1, t: 'note', text: '# Redis 推送事件 → PA 的 PSubscribe 收到' },
  { s: 2, t: 'pe', text: `pe-1   | DECIDE user=alice device= ip=172.18.0.5 resource=a -> allow=false (${reason})` },
  { s: 3, t: 'note', text: '# PA：SREM grants:<session> a · XADD zt:events（revoke，含 8 項 checks）' },
  { s: 3, t: 'pa', text: `pa-1   | REVOKE session=${sid} resource=a reason="${reason}"` },
  { s: 3, t: 'pa', text: 'pa-1   | REEVAL [PIP 變動 ti:bad_users (sadd)] checked=1 revoked=1' },
  { s: 3, t: 'pep', text: `pep-1  | REVOKE session=${sid} resource=a reason="${reason}"` },
  { s: 4, t: 'pa', text: `pa-1   | ACCESS session=${sid} user=alice resource=a -> allow=false (${reason})` },
  { s: 4, t: 'note', text: '# 瀏覽器收到 403「存取被拒絕」' },
]

const visibleLogs = computed(() => logs.filter(l => l.s <= props.step))
</script>

<template>
  <div>
    <div class="track">
      <template v-for="(st, i) in stations" :key="i">
        <div
          class="station"
          :class="{ on: i <= props.step, now: i === props.step }"
          :style="{ '--c': st.color }"
        >
          <div class="idx">{{ i + 1 }}</div>
          <div class="title">{{ st.title }}</div>
          <div class="body">{{ st.body }}</div>
          <code class="code">{{ st.code }}</code>
        </div>
        <div v-if="i < stations.length - 1" class="link" :class="{ on: i < props.step }">
          <span class="dot" />
        </div>
      </template>
    </div>

    <div class="term">
      <TransitionGroup name="log">
        <div v-for="l in visibleLogs" :key="l.text" class="line" :class="l.t">{{ l.text }}</div>
      </TransitionGroup>
    </div>
  </div>
</template>

<style scoped>
.track {
  display: flex;
  align-items: stretch;
}

.station {
  flex: 1;
  min-width: 0;
  padding: 0.55rem 0.6rem;
  border-radius: 10px;
  border: 1.5px solid color-mix(in srgb, var(--zt-line) 40%, transparent);
  opacity: 0.35;
  transition: opacity 0.35s, border-color 0.35s, background 0.35s, transform 0.35s;
}

.station.on {
  opacity: 1;
  border-color: var(--c);
}

.station.now {
  background: color-mix(in srgb, var(--c) 16%, transparent);
  transform: translateY(-4px);
}

.idx {
  font-family: ui-monospace, monospace;
  font-size: 0.75rem;
  color: var(--c);
  font-weight: 700;
}

.title {
  font-weight: 700;
}

.body {
  font-size: 0.72rem;
  line-height: 1.35;
  color: var(--zt-muted);
  min-height: 2.9em;
}

.code {
  display: block;
  margin-top: 0.3rem;
  font-size: 0.66rem;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.link {
  position: relative;
  width: 1.6rem;
  align-self: center;
  height: 2px;
  background: color-mix(in srgb, var(--zt-line) 40%, transparent);
}

.link .dot {
  position: absolute;
  top: -4px;
  left: 0;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: var(--zt-ctrl);
  opacity: 0;
}

.link.on {
  background: var(--zt-ctrl);
}

.link.on .dot {
  animation: travel 1.2s ease-in-out infinite;
}

@keyframes travel {
  0% { left: -4px; opacity: 0; }
  20% { opacity: 1; }
  80% { opacity: 1; }
  100% { left: calc(100% - 6px); opacity: 0; }
}

.term {
  margin-top: 1.1rem;
  padding: 0.7rem 0.9rem;
  border-radius: 10px;
  background: #0d1117;
  color: #c9d1d9;
  font-family: ui-monospace, 'JetBrains Mono', monospace;
  font-size: 0.7rem;
  line-height: 1.6;
  min-height: 11.5rem;
}

.line {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.line.cmd { color: #e6edf3; font-weight: 700; }
.line.out { color: #8b949e; }
.line.note { color: #7ee787; }
.line.pe { color: #79c0ff; }
.line.pa { color: #d2a8ff; }
.line.pep { color: #ffa657; }

.log-enter-active {
  transition: opacity 0.35s, transform 0.35s;
}

.log-enter-from {
  opacity: 0;
  transform: translateX(-10px);
}
</style>
