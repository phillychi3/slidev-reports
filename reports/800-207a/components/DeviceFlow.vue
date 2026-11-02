<script setup lang="ts">
import type { SeqActor, SeqMsg } from './SeqDiagram.vue'

// 受管理裝置入口（zt-demo docs/report.md §4.2）：gateway 驗證裝置憑證，經 XFCC 交給 pep-managed；
// PEP 只在 XFCC 最後一段是 gateway 的 SPIFFE ID 時採信（cmd/pep/mesh.go meshIdentity）
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })

const actors: SeqActor[] = [
  { id: 'd', label: '受管理電腦', color: 'var(--zt-res)' },
  { id: 'g', label: 'ingress gateway', color: 'var(--zt-pep)' },
  { id: 'p', label: 'pep-managed', color: 'var(--zt-pep)' },
  { id: 'e', label: 'PE', color: 'var(--zt-ctrl)' },
  { id: 'r', label: 'Redis（CDM）', color: 'var(--zt-pip)' },
]

const msgs: SeqMsg[] = [
  { g: 1, from: 'd', to: 'g', text: 'TLS（device.<網域>）出示 laptop1 憑證' },
  { g: 1, from: 'g', text: 'MUTUAL：以裝置 CA 驗證憑證鏈；沒有憑證 → 握手失敗' },
  { g: 2, from: 'g', note: true, text: 'SANITIZE_SET：丟掉用戶端自帶的 XFCC，只寫入驗證過的 Hash · Subject' },
  { g: 2, from: 'g', to: 'p', text: 'mTLS（來源 sa/zt-gateway）＋ XFCC' },
  { g: 3, from: 'p', to: 'e', note: true, text: 'XFCC =［Hash=…; Subject="CN=laptop1"］,［URI=spiffe://…/ns/zt-gateway/sa/zt-gateway］' },
  { g: 3, from: 'p', text: '最後一段不是 gateway → 拒絕；是 → 取 CN 與指紋' },
  { g: 4, from: 'p', to: 'e', text: 'AccessContext（device_id=laptop1, device_fp=…）' },
  { g: 4, from: 'e', to: 'r', text: 'GET posture:laptop1' },
  { g: 4, from: 'e', to: 'p', text: '依 CDM 狀態允許／拒絕', tone: 'allow' },
]

const captions = [
  '<b>裝置憑證在邊界驗證</b>：gateway 的 <code>tls.mode: MUTUAL</code>，憑證由管理者電腦上的裝置 CA 簽發',
  '<b>XFCC 不能偽造</b>：gateway 先清掉用戶端送來的 XFCC；公開入口帶假 XFCC，仍是「未受管理裝置」',
  '<b>PEP 只信 gateway</b>：sidecar 會在 XFCC 最後附上上一跳的 SPIFFE ID，不是 gateway 就不採信裝置與來源 IP',
  '<b>之後與 800-207 相同</b>：PE 第 6、7 項檢查裝置身分與 CDM；每個請求的指紋必須等於登入時的指紋',
]
</script>

<template>
  <SeqDiagram :actors="actors" :msgs="msgs" :captions="captions" :step="props.step" :height="320" />
</template>
