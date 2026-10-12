<script setup lang="ts">
import { onBeforeUnmount, ref, watch } from 'vue'

// 依 cmd/pe/main.go 的 decide() 順序模擬 criteria-based trust algorithm。
// 拒絕理由文字與 PE 原始碼一致。
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })

type R = 'pass' | 'fail' | 'skip'
interface Scenario {
  title: string
  user: string
  device: string
  resource: string
  results: R[]
  details: string[]
  reason: string
}

const checks = ['身分與效期', '威脅情資', '裝置身分與狀態', '資源角色']

const scenarios: Scenario[] = [
  {
    title: '借用電腦 → A',
    user: 'alice', device: '未受管理裝置', resource: 'A 一般資源',
    results: ['pass', 'pass', 'pass', 'pass'],
    details: ['passkey 登入 · session 未過期', 'ti:bad_users / ti:bad_ips 未命中', '無憑證，但 A 設定 allow_unmanaged: true', 'A 不要求角色'],
    reason: '使用者已驗證；政策允許未受管理裝置存取此資源',
  },
  {
    title: '借用電腦 → B',
    user: 'bob（admin）', device: '未受管理裝置', resource: 'B 敏感資源',
    results: ['pass', 'pass', 'fail', 'skip'],
    details: ['passkey 登入 · session 未過期', '未命中', '無憑證，B 的 allow_unmanaged: false', ''],
    reason: '此資源需要受管理裝置，請使用已配置裝置憑證的客戶端',
  },
  {
    title: '帳號列入風險名單',
    user: 'alice', device: '未受管理裝置', resource: 'A 一般資源',
    results: ['pass', 'fail', 'skip', 'skip'],
    details: ['passkey 登入 · session 未過期', 'SISMEMBER ti:bad_users alice → 1', '', ''],
    reason: '使用者帳號在威脅情資名單中',
  },
  {
    title: '受管理裝置 + alice → B',
    user: 'alice', device: 'laptop1（compliant）', resource: 'B 敏感資源',
    results: ['pass', 'pass', 'pass', 'fail'],
    details: ['passkey 登入 · session 未過期', '未命中', 'mTLS 憑證 CN=laptop1 · posture = compliant', 'B 要求 admin，alice 沒有'],
    reason: '缺少必要角色 [admin]',
  },
  {
    title: '受管理裝置 + bob → B',
    user: 'bob（admin）', device: 'laptop1（compliant）', resource: 'B 敏感資源',
    results: ['pass', 'pass', 'pass', 'pass'],
    details: ['passkey 登入 · session 未過期', '未命中', 'mTLS 憑證 CN=laptop1 · posture = compliant', 'bob 具有 admin'],
    reason: '所有條件皆符合',
  },
  {
    title: '裝置變成不合規',
    user: 'bob（admin）', device: 'laptop1（noncompliant）', resource: 'A 一般資源',
    results: ['pass', 'pass', 'fail', 'skip'],
    details: ['passkey 登入 · session 未過期', '未命中', 'GET posture:laptop1 → noncompliant', ''],
    reason: '裝置狀態為 "noncompliant"，要求 "compliant"',
  },
]

const current = ref(0)
const revealed = ref(0) // 已顯示結果的檢查數；checks.length + 1 代表顯示最終判定
let timers: ReturnType<typeof setTimeout>[] = []

function run(i: number) {
  timers.forEach(clearTimeout)
  timers = []
  current.value = Math.min(Math.max(i, 0), scenarios.length - 1)
  revealed.value = 0
  for (let k = 1; k <= checks.length + 1; k++)
    timers.push(setTimeout(() => (revealed.value = k), k * 380))
}

watch(() => props.step, s => run(s), { immediate: true })
onBeforeUnmount(() => timers.forEach(clearTimeout))

const icon: Record<R, string> = { pass: '✓', fail: '✕', skip: '—' }
</script>

<template>
  <div class="pe grid grid-cols-[13rem_1fr] gap-5">
    <!-- 情境選單：簡報時按空白鍵切換，也可以直接點 -->
    <div class="flex flex-col gap-1.5">
      <button
        v-for="(s, i) in scenarios" :key="i"
        class="scenario"
        :class="{ sel: i === current }"
        @click.stop="run(i)"
      >
        {{ s.title }}
      </button>
      <div class="text-xs mt-2 opacity-70 leading-snug">
        任何 PIP 查詢失敗<br>一律 <b>fail closed</b>（拒絕）
      </div>
    </div>

    <div>
      <div class="request">
        <span class="chip">PEP</span><span class="arrow">→</span>
        <span class="chip ctrl">PA</span><span class="arrow">→</span>
        <span class="chip ctrl">PE /decide</span>
        <span class="ml-3 font-mono text-sm">
          user=<b>{{ scenarios[current].user }}</b>
          · device=<b>{{ scenarios[current].device }}</b>
          · resource=<b>{{ scenarios[current].resource }}</b>
        </span>
      </div>

      <div class="flex flex-col gap-2 mt-3">
        <div
          v-for="(c, k) in checks" :key="c"
          class="check"
          :class="revealed > k ? scenarios[current].results[k] : 'pending'"
        >
          <div class="num">{{ k + 1 }}</div>
          <div class="name">{{ c }}</div>
          <div class="detail">{{ revealed > k ? scenarios[current].details[k] : '' }}</div>
          <div class="icon">{{ revealed > k ? icon[scenarios[current].results[k]] : '…' }}</div>
        </div>
      </div>

      <Transition name="verdict">
        <div
          v-if="revealed > checks.length"
          :key="current"
          class="verdict"
          :class="scenarios[current].results.includes('fail') ? 'deny' : 'allow'"
        >
          <b>{{ scenarios[current].results.includes('fail') ? '403 拒絕' : '允許，轉發到資源' }}</b>
          <span class="font-mono text-sm ml-2">reason: {{ scenarios[current].reason }}</span>
        </div>
      </Transition>
    </div>
  </div>
</template>

<style scoped>
.scenario {
  text-align: left;
  font-size: 0.85rem;
  padding: 0.4rem 0.7rem;
  border-radius: 8px;
  border: 1px solid color-mix(in srgb, var(--zt-line) 50%, transparent);
  transition: background 0.2s, border-color 0.2s;
}

.scenario:hover {
  background: color-mix(in srgb, var(--zt-ctrl) 10%, transparent);
}

.scenario.sel {
  border-color: var(--zt-ctrl);
  background: color-mix(in srgb, var(--zt-ctrl) 18%, transparent);
  font-weight: 700;
}

.request {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 0.3rem;
}

.chip {
  font-size: 0.8rem;
  font-weight: 700;
  padding: 0.1rem 0.55rem;
  border-radius: 999px;
  border: 1.5px solid var(--zt-pep);
}

.chip.ctrl {
  border-color: var(--zt-ctrl);
}

.arrow {
  color: var(--zt-muted);
}

.check {
  display: grid;
  grid-template-columns: 1.8rem 8.5rem 1fr 2rem;
  align-items: center;
  padding: 0.5rem 0.75rem;
  border-radius: 10px;
  border: 1.5px solid color-mix(in srgb, var(--zt-line) 40%, transparent);
  transition: background 0.3s, border-color 0.3s, opacity 0.3s;
}

.check .num {
  font-family: ui-monospace, monospace;
  color: var(--zt-muted);
}

.check .name {
  font-weight: 700;
}

.check .detail {
  font-size: 0.85rem;
  font-family: ui-monospace, monospace;
  color: var(--zt-muted);
}

.check .icon {
  text-align: center;
  font-weight: 800;
  font-size: 1.15rem;
}

.check.pending {
  opacity: 0.45;
}

.check.pass {
  border-color: var(--zt-allow);
  background: color-mix(in srgb, var(--zt-allow) 10%, transparent);
}

.check.pass .icon {
  color: var(--zt-allow);
}

.check.fail {
  border-color: var(--zt-deny);
  background: color-mix(in srgb, var(--zt-deny) 14%, transparent);
  animation: shake 0.35s;
}

.check.fail .icon {
  color: var(--zt-deny);
}

.check.skip {
  opacity: 0.35;
}

.verdict {
  margin-top: 0.8rem;
  padding: 0.6rem 0.9rem;
  border-radius: 10px;
  border-left: 5px solid;
}

.verdict.allow {
  border-color: var(--zt-allow);
  background: color-mix(in srgb, var(--zt-allow) 14%, transparent);
}

.verdict.deny {
  border-color: var(--zt-deny);
  background: color-mix(in srgb, var(--zt-deny) 14%, transparent);
}

.verdict-enter-active {
  transition: opacity 0.3s, transform 0.3s;
}

.verdict-enter-from {
  opacity: 0;
  transform: translateY(8px);
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-5px); }
  75% { transform: translateX(5px); }
}
</style>
