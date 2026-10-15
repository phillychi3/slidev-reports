<script setup lang="ts">
import type { SeqActor, SeqMsg } from './SeqDiagram.vue'

// OIDC Authorization Code + PKCE + passkey（docs/report.md §3.1，對應 cmd/pep login / callback）
const props = withDefaults(defineProps<{ step?: number }>(), { step: 0 })

const actors: SeqActor[] = [
  { id: 'b', label: '借用電腦瀏覽器', color: 'var(--zt-res)' },
  { id: 'p', label: 'PEP', color: 'var(--zt-pep)' },
  { id: 'k', label: 'Keycloak', color: 'var(--zt-idp)' },
  { id: 'm', label: '手機 passkey', color: 'var(--zt-idp)' },
]

const msgs: SeqMsg[] = [
  { g: 1, from: 'b', to: 'p', text: 'GET /a/（沒有 session）' },
  { g: 1, from: 'p', to: 'b', text: '302 /_zt/login?return=/a/' },
  { g: 1, from: 'b', to: 'p', text: 'GET /_zt/login' },
  { g: 1, from: 'p', note: true, text: '產生 state · nonce · PKCE verifier，暫存在 PEP' },
  { g: 1, from: 'p', to: 'b', text: '302 授權端點 · Set-Cookie __Host-zt_login=state' },
  { g: 2, from: 'b', to: 'k', text: 'GET /auth?client_id=pep · code_challenge(S256) · nonce · prompt=login' },
  { g: 2, from: 'k', to: 'b', text: 'Username Form → WebAuthn challenge（RP ID = 展示網域）' },
  { g: 3, from: 'b', to: 'm', text: '跨裝置：QR code + BLE 近距離', dashed: true },
  { g: 3, from: 'm', text: '使用者驗證（指紋／臉／PIN）→ 私鑰簽章' },
  { g: 3, from: 'm', to: 'k', text: 'assertion（經瀏覽器送回）', dashed: true },
  { g: 3, from: 'k', text: '用註冊時的公鑰驗章 ✓' },
  { g: 4, from: 'k', to: 'b', text: '302 /_zt/callback?code&state' },
  { g: 4, from: 'b', to: 'p', text: 'GET /_zt/callback（帶 __Host-zt_login）' },
  { g: 4, from: 'p', note: true, text: '__Host-zt_login cookie 必須等於 state，且未超過 10 分鐘' },
  { g: 5, from: 'p', to: 'k', text: 'POST token：client secret + code_verifier（容器內 backchannel）' },
  { g: 5, from: 'k', to: 'p', text: 'ID token + access token' },
  { g: 5, from: 'p', note: true, text: 'JWKS 驗簽 · iss · aud=pep · nonce · azp=pep · 兩個 token 的 sub 一致' },
  { g: 5, from: 'p', to: 'b', text: 'Set-Cookie __Host-zt_session · 302 /a/', tone: 'allow' },
]

const captions = [
  '<b>PEP 發起 OIDC 授權碼流程</b>：state 綁定這個瀏覽器（防 login CSRF），PKCE 防授權碼被攔截',
  '<b>Keycloak 只問帳號</b>：<code>prompt=login</code> 每次都要互動驗證，整個流程沒有密碼欄位',
  '<b>生物辨識只在手機本地</b>：私鑰不離開手機，伺服器只用公鑰驗章',
  '<b>callback 檢查</b>：登入 cookie 對上 state，確保完成登入的是發起登入的同一個瀏覽器',
  '<b>Session 效期</b> = min(access token, ID token, 15 分鐘)；cookie 為 <code>Secure · HttpOnly · SameSite=Lax</code>',
]
</script>

<template>
  <SeqDiagram :actors="actors" :msgs="msgs" :captions="captions" :step="props.step" :height="440" />
</template>
