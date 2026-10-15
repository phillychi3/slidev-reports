<script setup lang="ts">
import { computed, onBeforeUnmount, ref, watch } from 'vue'

// 依 cmd/pe/main.go 的 decide() 模擬 criteria-based trust algorithm（§3.3.1）：
// 8 項條件全部評估並列出，第一個失敗的條件就是拒絕原因。條件名稱與文字與 PE 原始碼一致。
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })

type R = 'pass' | 'fail' | 'skip'
interface Check { name: string, src: string, result: R, detail: string }

const policy: Record<string, { desc: string, admin: boolean, unmanaged: boolean }> = {
  a: { desc: '一般資源：已驗證使用者，可使用未受管理裝置', admin: false, unmanaged: true },
  b: { desc: '敏感資源：需要 admin 角色及合規受管理裝置', admin: true, unmanaged: false },
  c: { desc: '管理資源：需要 admin 角色，可使用未受管理裝置', admin: true, unmanaged: true },
}

// 展示只用借用電腦（無裝置憑證），所以裝置狀態（CDM）一律略過
interface Scenario {
  title: string
  user: 'alice' | 'bob'
  res: 'a' | 'b' | 'c'
  badUser?: boolean
}

const scenarios: Scenario[] = [
  { title: 'alice · 借用電腦 → a', user: 'alice', res: 'a' },
  { title: 'alice · 借用電腦 → b', user: 'alice', res: 'b' },
  { title: 'bob · 借用電腦 → c', user: 'bob', res: 'c' },
  { title: 'bob · 借用電腦 → b', user: 'bob', res: 'b' },
  { title: 'alice 列入風險名單 → a', user: 'alice', res: 'a', badUser: true },
]

function decide(s: Scenario) {
  const rp = policy[s.res]
  const ip = '172.18.0.5'
  const isAdmin = s.user === 'bob'
  const c: Check[] = [
    { name: '資源政策', src: 'policy.json', result: 'pass', detail: rp.desc },
    { name: '使用者身分', src: 'subject', result: 'pass', detail: s.user },
    { name: '登入效期', src: 'expires_at', result: 'pass', detail: '有效至 14:18:05' },
    { name: '威脅情資：來源 IP', src: 'ti:bad_ips', result: 'pass', detail: `"${ip}" 不在 ti:bad_ips` },
    s.badUser
      ? { name: '威脅情資：帳號', src: 'ti:bad_users', result: 'fail', detail: '使用者帳號在威脅情資名單中' }
      : { name: '威脅情資：帳號', src: 'ti:bad_users', result: 'pass', detail: `"${s.user}" 不在 ti:bad_users` },
  ]
  c.push(rp.unmanaged
    ? { name: '裝置身分', src: 'device_id / fp', result: 'pass', detail: '未受管理裝置；政策允許此資源' }
    : { name: '裝置身分', src: 'device_id / fp', result: 'fail', detail: '此資源需要受管理裝置，請使用已配置裝置憑證的客戶端' })
  c.push({ name: '裝置狀態（CDM）', src: 'posture:<CN>', result: 'skip', detail: '沒有裝置憑證' })
  if (!rp.admin)
    c.push({ name: '角色', src: 'roles', result: 'pass', detail: '任何已驗證使用者' })
  else if (isAdmin)
    c.push({ name: '角色', src: 'roles', result: 'pass', detail: '具備 [admin] 之一' })
  else
    c.push({ name: '角色', src: 'roles', result: 'fail', detail: '缺少必要角色 [admin]' })

  const failed = c.find(x => x.result === 'fail')
  return { checks: c, allow: !failed, reason: failed ? failed.detail : '使用者已驗證；政策允許未受管理裝置存取此資源' }
}

const current = ref(0)
const revealed = ref(0) // 已顯示的條件數；9 代表顯示最終判定
const result = computed(() => decide(scenarios[current.value]))
let timers: ReturnType<typeof setTimeout>[] = []

function run(i: number) {
  timers.forEach(clearTimeout)
  timers = []
  current.value = Math.min(Math.max(i, 0), scenarios.length - 1)
  revealed.value = 0
  for (let k = 1; k <= 9; k++)
    timers.push(setTimeout(() => (revealed.value = k), k * 200))
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
        <span class="ml-3 font-mono text-xs">
          user=<b>{{ scenarios[current].user }}</b>
          · device=<b>未受管理裝置</b>
          · resource=<b>{{ scenarios[current].res }}</b>
        </span>
      </div>

      <div class="flex flex-col gap-1 mt-2">
        <div
          v-for="(c, k) in result.checks" :key="c.name"
          class="check"
          :class="revealed > k ? c.result : 'pending'"
        >
          <div class="num">{{ k + 1 }}</div>
          <div class="name">{{ c.name }}</div>
          <div class="src">{{ c.src }}</div>
          <div class="detail">{{ revealed > k ? c.detail : '' }}</div>
          <div class="icon">{{ revealed > k ? icon[c.result] : '…' }}</div>
        </div>
      </div>

      <Transition name="verdict">
        <div
          v-if="revealed > result.checks.length"
          :key="current"
          class="verdict"
          :class="result.allow ? 'allow' : 'deny'"
        >
          <b>{{ result.allow ? '允許，轉發到資源' : '403 拒絕' }}</b>
          <span class="font-mono text-xs ml-2">reason（第一個失敗的條件）: {{ result.reason }}</span>
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
  grid-template-columns: 1.4rem 8.6rem 6.2rem 1fr 1.6rem;
  align-items: center;
  padding: 0.22rem 0.7rem;
  font-size: 0.85rem;
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

.check .src {
  font-size: 0.7rem;
  font-family: ui-monospace, monospace;
  color: var(--zt-pip);
}

.check .detail {
  font-size: 0.75rem;
  font-family: ui-monospace, monospace;
  color: var(--zt-muted);
}

.check .icon {
  text-align: center;
  font-weight: 800;
  font-size: 1rem;
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
  margin-top: 0.5rem;
  padding: 0.45rem 0.9rem;
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
