<script setup lang="ts">
import type { SeqActor, SeqMsg } from './SeqDiagram.vue'

// 每次資源請求（zt-demo docs/report.md §3.2）：PEP → PA → PE 沿用 800-207，
// 新增的是每一跳的 mTLS 與 PEP 簽發、資源端 sidecar 驗證的 X-ZT-Assertion
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })

const actors: SeqActor[] = [
  { id: 'b', label: '瀏覽器', color: 'var(--zt-res)' },
  { id: 'p', label: 'PEP', color: 'var(--zt-pep)' },
  { id: 'a', label: 'PA', color: 'var(--zt-ctrl)' },
  { id: 'e', label: 'PE', color: 'var(--zt-ctrl)' },
  { id: 'r', label: 'Redis（PIP）', color: 'var(--zt-pip)' },
  { id: 's', label: 'sidecar → Lithium', color: 'var(--zt-res)' },
]

const msgs: SeqMsg[] = [
  { g: 1, from: 'b', to: 'p', text: 'GET /media/api/…（經 gateway）' },
  { g: 1, from: 'p', note: true, text: 'XFCC 上一跳 = gateway？查 session、比對裝置指紋' },
  { g: 2, from: 'p', to: 'a', text: 'POST /access（mTLS · 來源 sa/pep）' },
  { g: 2, from: 'a', to: 'e', text: 'POST /decide（mTLS · 來源 sa/pa）' },
  { g: 3, from: 'e', to: 'r', text: 'SISMEMBER ti:bad_ips / ti:bad_users' },
  { g: 3, from: 'e', to: 'a', text: 'Decision { allow, reason, checks[8] }' },
  { g: 3, from: 'a', to: 'r', text: 'SADD／SREM grants · XADD zt:events' },
  { g: 3, from: 'a', to: 'p', text: 'Decision' },
  { g: 4, from: 'p', to: 'b', text: '拒絕：403 拒絕頁，顯示 reason', tone: 'deny' },
  { g: 4, from: 'p', note: true, text: '允許：簽發 X-ZT-Assertion（ES256 · aud=media · 60 秒）' },
  { g: 4, from: 'p', to: 's', text: '轉發；刪 Cookie / Authorization / XFCC，加 X-ZT-User + X-ZT-Assertion', tone: 'allow' },
  { g: 5, from: 's', note: true, text: '驗證來源 sa/pep 與聲明' },
  { g: 5, from: 's', to: 'b', text: '200 回應（經 PEP）', tone: 'allow' },
]

const captions = [
  '<b>gateway → PEP 也是 mTLS</b>：PEP 只在 XFCC 最後一段是 gateway 的 SPIFFE ID 時，才採信裝置憑證與來源 IP',
  '<b>控制平面每一跳都經 sidecar</b>：雙向 SPIFFE 驗證，再由 AuthorizationPolicy 限制「只有 pa 能 POST /decide」',
  '<b>判斷沿用 800-207</b>：PE 評估 8 項條件、PIP 查不到就 fail closed；PA 記錄 grants 與稽核',
  '<b>新增：使用者聲明</b> —— 不再只靠 <code>X-ZT-User</code>，PEP 對每個請求簽一張 60 秒、綁定資源的 JWT',
  '<b>資源端再驗一次</b>：sidecar 以 <code>RequestAuthentication</code> 驗聲明；nginx 轉給後端時，後端的 sidecar 再驗一次',
]
</script>

<template>
  <SeqDiagram :actors="actors" :msgs="msgs" :captions="captions" :step="props.step" :height="420" />
</template>
