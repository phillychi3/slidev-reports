<script setup lang="ts">
import type { SeqActor, SeqMsg } from './SeqDiagram.vue'

// 每次資源請求的資料交換（docs/report.md §3.2，對應 pep.ServeHTTP → pa.handleAccess → pe.decide）
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })

const actors: SeqActor[] = [
  { id: 'b', label: '瀏覽器', color: 'var(--zt-res)' },
  { id: 'p', label: 'PEP', color: 'var(--zt-pep)' },
  { id: 'a', label: 'PA', color: 'var(--zt-ctrl)' },
  { id: 'e', label: 'PE', color: 'var(--zt-ctrl)' },
  { id: 'r', label: 'Redis（PIP）', color: 'var(--zt-pip)' },
  { id: 's', label: '資源 B', color: 'var(--zt-res)' },
]

const msgs: SeqMsg[] = [
  { g: 1, from: 'b', to: 'p', text: 'GET /b/…（帶 __Host-zt_session）' },
  { g: 1, from: 'p', note: true, text: '查 session；沒有或已過期 → 導向登入，不問 PA' },
  { g: 2, from: 'p', to: 'a', text: 'POST /access　AccessContext' },
  { g: 2, from: 'a', to: 'e', text: 'POST /decide　AccessContext' },
  { g: 3, from: 'e', to: 'r', text: 'SISMEMBER ti:bad_ips / ti:bad_users' },
  { g: 3, from: 'e', note: true, text: '依序評估 8 項條件，全部記進 checks[]' },
  { g: 3, from: 'e', to: 'a', text: 'Decision { allow, reason, checks[8] }' },
  { g: 4, from: 'a', to: 'r', text: '允許：SET session · SADD grants／拒絕：SET session · SREM grants' },
  { g: 4, from: 'a', to: 'r', text: 'XADD zt:events（稽核，含 checks）' },
  { g: 4, from: 'a', to: 'p', text: 'Decision' },
  { g: 5, from: 'p', to: 's', text: '允許：轉發；刪 Cookie / Authorization，加 X-ZT-User / X-ZT-Device', tone: 'allow' },
  { g: 5, from: 's', to: 'b', text: '200 回應（經 PEP）', tone: 'allow' },
  { g: 5, from: 'p', to: 'b', text: '拒絕：403 拒絕頁，顯示 reason', tone: 'deny' },
]

const captions = [
  '<b>PEP 先擋一層</b>：沒有有效 session 就導向 passkey 登入，連 PA 都不問',
  '<b>AccessContext</b> 原封不動地 PEP → PA → PE；PE 不保存狀態，只負責判斷',
  '<b>PE 讀 PIP</b>：威脅情資從 Redis 即時查，查不到就 fail closed',
  '<b>PA 記錄授權</b>：之後要撤銷就靠 grants；每一筆決策都進稽核 stream',
  '<b>PEP 不快取授權</b>：每一個請求都重新走 PA → PE；資源只看得到 X-ZT-User，拿不到 cookie',
]
</script>

<template>
  <SeqDiagram :actors="actors" :msgs="msgs" :captions="captions" :step="props.step" :height="420" />
</template>
