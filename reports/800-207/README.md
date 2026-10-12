# 無密碼零信任存取（NIST SP 800-207）

約 4 分鐘的專題簡報，介紹 zt-demo：Go 實作的 PEP / PA / PE，搭配 Keycloak passkey 無密碼登入。

- `pnpm run dev` 後開 <http://localhost:3030>，按 `p` 進簡報者模式可看逐頁講稿與計時
- 動畫元件在 `components/`：`ZtModel`（800-207 模型）、`ArchDiagram`（系統架構）、`KeyPair`（passkey 原理）、`LoginFlow`（登入循序圖）、`PolicyEngine`（PE 決策，可直接點情境）、`RevokeFlow`（撤銷流程）
- 動畫跟著點擊推進，每頁的 `clicks:` 需與元件步數一致
