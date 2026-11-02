<script setup lang="ts">
import { computed, ref, watch } from 'vue'

// 叢集內實測（zt-demo docs/report.md §9）：同一個請求依序經過網路層（Dataplane V2）與身分層（sidecar），
// 標出是哪一層擋下。按空白鍵切換情境，也可以直接點左側清單。
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })

type R = 'pass' | 'fail' | 'skip'

interface Scenario {
  title: string
  src: string
  srcSub: string
  dst: string
  dstSub: string
  req: string
  net: [R, string]
  id: [R, string]
  result: string
  by: string
  moral: string
}

const scenarios: Scenario[] = [
  {
    title: '正常路徑：PEP 帶聲明',
    src: 'pep', srcSub: 'sa/pep', dst: 'lithium', dstSub: 'zt-media',
    req: '+ X-ZT-Assertion（aud=media）',
    net: ['pass', 'zt-system 的 PEP → lithium:80，允許'],
    id: ['pass', '來源 sa/pep ✓ · 聲明簽章、aud、效期 ✓'],
    result: '200', by: '—',
    moral: '兩層都放行，才到得了應用',
  },
  {
    title: '被攻陷的 nginx 偽造使用者',
    src: 'lithium', srcSub: '被攻陷', dst: 'lithium-api', dstSub: 'zt-media',
    req: 'X-ZT-User: bob（沒有聲明）',
    net: ['pass', '同 namespace、lithium → lithium-api:8000，允許'],
    id: ['fail', 'requestPrincipals 為空：沒有 PEP 簽發的聲明'],
    result: '403', by: '身分層',
    moral: '只看 X-ZT-User 的資源，這一步就會被騙過',
  },
  {
    title: '標籤冒充 PE 連 Redis',
    src: 'test pod', srcSub: 'sa/default · app=pe', dst: 'redis', dstSub: 'zt-system',
    req: 'TCP 6379',
    net: ['pass', '標籤 app=pe 符合規則 → 放行（被騙過）'],
    id: ['fail', 'SPIFFE 是 sa/default，不在 Redis 允許清單'],
    result: '連線被關閉', by: '身分層',
    moral: '標籤騙得過網路層，騙不過 SPIFFE 身分',
  },
  {
    title: 'PE 誤判也擋得住：沒有 admin',
    src: 'pep', srcSub: 'sa/pep', dst: 'yass', dstSub: 'zt-status',
    req: '+ X-ZT-Assertion（aud=status，roles 無 admin）',
    net: ['pass', 'zt-system 的 PEP → yass，允許'],
    id: ['fail', 'request.auth.claims[roles] 不含 admin'],
    result: '403', by: '身分層',
    moral: '縱深防禦：資源端獨立檢查使用者聲明',
  },
  {
    title: '繞過 sidecar',
    src: '無 sidecar 的 pod', srcSub: 'zt-media', dst: 'lithium-api', dstSub: 'zt-media',
    req: '明文 HTTP',
    net: ['fail', '來源 pod 不在 ingress 允許清單 → 丟棄'],
    id: ['skip', '封包到不了 sidecar（STRICT 也不收明文）'],
    result: '逾時', by: '網路層',
    moral: '身分層被繞過時，網路層仍在',
  },
  {
    title: '橫向移動到另一個應用',
    src: 'lithium', srcSub: 'zt-media', dst: 'yass', dstSub: 'zt-status',
    req: 'GET /',
    net: ['fail', 'zt-media 出站只允許同 namespace'],
    id: ['fail', 'Sidecar REGISTRY_ONLY：yass 不在出站清單'],
    result: '連線被重設', by: '兩層',
    moral: '微分段：兩層各自都擋得住',
  },
]

const cur = ref(props.step)
watch(() => props.step, s => (cur.value = s))
const s = computed(() => scenarios[Math.min(cur.value, scenarios.length - 1)])
const ok = computed(() => s.value.by === '—')
const mark: Record<R, string> = { pass: '✓', fail: '✗', skip: '—' }
</script>

<template>
  <div class="probe">
    <ol class="list">
      <li
        v-for="(sc, i) in scenarios" :key="i"
        :class="{ now: i === cur }"
        @click="cur = i"
      >
        <span class="n">{{ i + 1 }}</span>{{ sc.title }}
      </li>
    </ol>

    <div class="stage">
      <div class="path">
        <div class="card">
          <div class="name">{{ s.src }}</div>
          <div class="sub">{{ s.srcSub }}</div>
        </div>
        <div class="wire" />
        <div class="gate" :class="s.net[0]">
          <div class="gtitle">網路層</div>
          <div class="gsub">NetworkPolicy · L3/L4</div>
          <div class="mark">{{ mark[s.net[0]] }}</div>
        </div>
        <div class="wire" :class="{ cut: s.net[0] !== 'pass' }" />
        <div class="gate" :class="s.id[0]">
          <div class="gtitle">身分層</div>
          <div class="gsub">sidecar · SPIFFE＋聲明</div>
          <div class="mark">{{ mark[s.id[0]] }}</div>
        </div>
        <div class="wire" :class="{ cut: !ok }" />
        <div class="card" :class="{ dim: !ok }">
          <div class="name">{{ s.dst }}</div>
          <div class="sub">{{ s.dstSub }}</div>
        </div>
      </div>

      <div class="req"><code>{{ s.req }}</code></div>

      <Transition name="cap" mode="out-in">
        <div :key="cur" class="detail">
          <div class="row"><b>網路層</b><span :class="s.net[0]">{{ s.net[1] }}</span></div>
          <div class="row"><b>身分層</b><span :class="s.id[0]">{{ s.id[1] }}</span></div>
          <div class="row">
            <b>結果</b>
            <span class="result" :class="ok ? 'pass' : 'fail'">{{ s.result }}</span>
            <span v-if="!ok" class="by">擋下：{{ s.by }}</span>
          </div>
          <div class="moral">{{ s.moral }}</div>
        </div>
      </Transition>
    </div>
  </div>
</template>

<style scoped>
.probe {
  display: grid;
  grid-template-columns: 15rem 1fr;
  gap: 1.4rem;
  align-items: start;
}

.list {
  list-style: none;
  padding: 0;
  margin: 0;
  font-size: 0.8rem;
}

.list li {
  margin: 0 !important;
  padding: 0.45rem 0.6rem;
  border-radius: 8px;
  cursor: pointer;
  opacity: 0.55;
  transition: opacity 0.25s, background 0.25s;
}

.list li.now {
  opacity: 1;
  background: color-mix(in srgb, var(--zt-ctrl) 16%, transparent);
}

.list .n {
  display: inline-block;
  width: 1.3rem;
  font-family: ui-monospace, monospace;
  color: var(--zt-ctrl);
  font-weight: 700;
}

.path {
  display: flex;
  align-items: center;
}

.card,
.gate {
  flex: none;
  text-align: center;
  border-radius: 10px;
  padding: 0.5rem 0.6rem;
  transition: opacity 0.3s, border-color 0.3s, background 0.3s;
}

.card {
  width: 7rem;
  border: 1.5px solid var(--zt-res);
  background: color-mix(in srgb, var(--zt-res) 14%, transparent);
}

.card.dim {
  opacity: 0.35;
}

.name {
  font-weight: 700;
  font-size: 0.9rem;
}

.sub,
.gsub {
  font-size: 0.66rem;
  color: var(--zt-muted);
}

.gsub {
  white-space: nowrap;
}

.gate {
  width: 9.2rem;
  border: 2px solid var(--zt-line);
}

.gate.pass {
  border-color: var(--zt-allow);
  background: color-mix(in srgb, var(--zt-allow) 12%, transparent);
}

.gate.fail {
  border-color: var(--zt-deny);
  background: color-mix(in srgb, var(--zt-deny) 14%, transparent);
}

.gate.skip {
  opacity: 0.45;
  border-style: dashed;
}

.gtitle {
  font-weight: 700;
  font-size: 0.85rem;
}

.mark {
  font-size: 1.3rem;
  font-weight: 800;
  line-height: 1.4;
}

.gate.pass .mark { color: var(--zt-allow); }
.gate.fail .mark { color: var(--zt-deny); }

.wire {
  flex: 1;
  min-width: 0.8rem;
  height: 2px;
  background: var(--zt-allow);
  transition: background 0.3s, opacity 0.3s;
}

.wire.cut {
  background: repeating-linear-gradient(90deg, var(--zt-line) 0 5px, transparent 5px 10px);
  opacity: 0.6;
}

.req {
  margin-top: 0.5rem;
  font-size: 0.75rem;
  color: var(--zt-muted);
}

.detail {
  margin-top: 1rem;
  font-size: 0.85rem;
}

.row {
  display: flex;
  gap: 0.8rem;
  align-items: baseline;
  padding: 0.25rem 0;
  border-bottom: 1px solid color-mix(in srgb, var(--zt-line) 30%, transparent);
}

.row b {
  flex: none;
  width: 3.2rem;
  color: var(--zt-muted);
  font-weight: 600;
}

.row .pass { color: var(--zt-allow); }
.row .fail { color: var(--zt-deny); }
.row .skip { color: var(--zt-muted); }

.result {
  font-family: ui-monospace, monospace;
  font-weight: 700;
}

.by {
  font-size: 0.75rem;
  color: var(--zt-muted);
}

.moral {
  margin-top: 0.8rem;
  font-weight: 700;
}

.cap-enter-active,
.cap-leave-active {
  transition: opacity 0.2s, transform 0.2s;
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
