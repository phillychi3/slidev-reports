---
theme: seriph
background: https://cover.sli.dev
title: 無密碼零信任存取
info: |
  ## 無密碼零信任存取
  依 NIST SP 800-207 核心原則實作的教學原型：Go 寫的 PEP / PA / PE，搭配 Keycloak passkey 登入。
class: text-center
colorSchema: dark
drawings:
  persist: false
transition: slide-left
mdc: true
duration: 6min
fonts:
  sans: Noto Sans TC
  serif: Noto Serif TC
  mono: JetBrains Mono
---

# 無密碼零信任存取

## 依 NIST SP 800-207 的教學原型

<div class="mt-8 opacity-80">
借一台電腦、只開瀏覽器、用自己的手機 passkey 登入
</div>

<div class="mt-10 text-sm opacity-60 font-mono">
Go（PEP · PA · PE）· Keycloak · Caddy · Redis · step-ca · Docker Compose
</div>

<div class="pt-10">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    開始 <carbon:arrow-right class="inline"/>
  </span>
</div>

<!--
（約 15 秒）
大家好，今天介紹的是一個依照 NIST SP 800-207 做的零信任存取原型。
情境是：借一台別人的電腦，不裝任何東西、不輸入任何密碼，只用自己手機上的 passkey 登入，然後系統要決定這台電腦能碰哪些資源。
這次依照上次會議的意見，補上了詳細架構和資料交換的細節。
-->

---
clicks: 3
---

# NIST SP 800-207：零信任架構

<div grid="~ cols-[3fr_2fr] gap-6" class="mt-4 items-center">
<div>
  <ZtModel :step="$clicks" />
</div>
<div class="text-[0.95rem] leading-relaxed">

- **不因網路位置給予信任**<br><span class="opacity-60 text-sm">每一次存取都要重新授權</span>

<div v-click="1">

- **控制／資料平面分離**<br><span class="opacity-60 text-sm">PEP 是資源唯一入口；PE 決策、PA 開關路徑</span>

</div>
<div v-click="2">

- **動態政策**<br><span class="opacity-60 text-sm">身分 + 裝置狀態 + 威脅情資一起判斷</span>

</div>
<div v-click="3">

- **本專案**<br><span class="opacity-60 text-sm">PEP / PA / PE 用 Go 自己寫，PIP 用現成套件</span>

</div>
</div>
</div>

<!--
（約 30 秒）
800-207 的核心是：不因為你在哪個網路就信任你，每一次存取都要重新判斷。

[click] 它把系統切成兩個平面。資料平面上，使用者只能透過 PEP 碰到資源；控制平面上，PE 負責做決策，PA 負責把決策變成「開」或「關」。

[click] PE 做決策時會參考各種 PIP：裝置狀態、威脅情資、存取政策、身分管理。

[click] 在我們的專案裡，PEP、PA、PE 是用 Go 自己實作的，身分交給 Keycloak，裝置狀態跟威脅情資放在 Redis，這部分是模擬資料。
-->

---
clicks: 4
class: compact
---

# 詳細架構圖

<ArchDetail :step="$clicks" />

<!--
（約 55 秒）
這是實際部署的元件和網段。左邊是客戶端，上面是對外的 untrusted 網段，下面是 internal 的控制平面，右邊三個資源各自在獨立的 enclave。Keycloak 和 PEP 跨在網段邊界上，代表同時接在兩個網段。

[click] 公開入口：瀏覽器用公開 CA 的 HTTPS 連到 Caddy。Caddy 再用 step-ca 簽的憑證連 PEP，而且會驗證主機名稱是 pep。Caddy 不轉送用戶端憑證，所以借用電腦一律是未受管理裝置。/auth 轉給 Keycloak。手機跟借用電腦之間是 WebAuthn 的跨裝置通道，不經過伺服器網路。

[click] 控制平面：每個請求 PEP 把 AccessContext 送給 PA，PA 轉給 PE，PE 到 Redis 查威脅情資。PA 把授權和稽核事件寫進 Redis，撤銷則透過 pub/sub 通知 PEP。PEP 也在內部跟 Keycloak 換 token、拿 JWKS。威脅情資由管理者用 redis-cli 寫入，是模擬資料。

[click] 資料平面：只有 PEP 能同時連到三個 enclave，它轉發時帶上 X-ZT-User。資源之間互相連不到，這就是微分段。

[click] 最後依連線保護等級上色：綠色是有 TLS 而且驗證對端；橘色是 PEP 到 Keycloak，雖然是 HTTP，但 token 本身有簽章；紅色是控制平面內部，目前只靠網段隔離，這是已知限制。
-->

---
class: dense
---

# 網段與連線保護

<div grid="~ cols-[1fr_1.35fr] gap-6" class="mt-2">
<div>

### 網段（Docker network）

| 網段 | internal | 成員 |
| --- | --- | --- |
| untrusted | 否 | Caddy、Keycloak、PEP |
| control-plane | **是** | Keycloak、step-ca、Redis、PE、PA、PEP |
| enclave-a | **是** | PEP、資源 a（+ 模擬被攻陷的 res-a-shell） |
| enclave-b | **是** | PEP、資源 b |
| enclave-c | **是** | PEP、資源 c |

<div class="text-xs opacity-75 mt-3 leading-relaxed">

- **唯一入口**：資源不接 untrusted，外部只能經 PEP
- **微分段**：在 res-a 的網路命名空間執行 `wget http://res-b` → 失敗

</div>
</div>
<div>

### 每條連線的保護

| 連線 | 協定 | 加密 | 對端驗證 |
| --- | --- | --- | --- |
| 瀏覽器 → Caddy | HTTPS | ✅ 公開 CA | 瀏覽器驗證伺服器 |
| Caddy → PEP | HTTPS | ✅ step-ca | 驗 PEP 憑證與主機名 `pep` |
| 手機 ↔ 借用電腦 | WebAuthn hybrid | ✅ | FIDO 協定 |
| PEP → Keycloak（token、JWKS） | HTTP | ❌ | token 有簽章，以 JWKS 驗證 |
| PEP → PA → PE | HTTP | ❌ | **無**，只靠 internal network |
| PA／PE／PEP → Redis | Redis | ❌ | **無** |
| PEP → 資源 | HTTP | ❌ | **無** |

</div>
</div>

<!--
（約 30 秒）
左邊是網段成員。標成 internal 的網段沒有對外路由，容器之間只能透過同時接在兩個網段的元件互通。資源不接 untrusted，所以外部只能經 PEP。

右邊是每條連線的保護。對外的連線都有 TLS；但控制平面內部，PEP、PA、PE、Redis 到資源，目前都是明文、沒有服務身分驗證。網路隔離不等於 800-207 要求的完整通訊保護，這是後續要補的內部 mTLS。
-->

---
clicks: 4
class: compact
---

# 資料交換 ①：登入（OIDC Code + PKCE + passkey）

<LoginFlow :step="$clicks" />

<!--
（約 50 秒）
第一段，PEP 發現沒有 session，先導向自己的 /_zt/login，在這裡產生 state、nonce、PKCE verifier，然後把 state 寫進登入 cookie，再導到 Keycloak。

[click] 授權請求帶 client_id、code_challenge、nonce，還有 prompt=login，每次都要互動驗證。Keycloak 只問帳號，接著發 WebAuthn challenge，綁定展示網域。

[click] 借用電腦顯示 QR code，手機掃描並用藍牙確認在附近；手機做使用者驗證後用私鑰簽章，Keycloak 用公鑰驗章。

[click] Keycloak 帶授權碼導回 PEP 的 callback。PEP 檢查登入 cookie 等於 state，確保完成登入的是同一個瀏覽器，這可以防 login CSRF。

[click] PEP 在容器內部用 client secret 加 code_verifier 換 token，驗簽章、iss、aud、nonce、azp，最後發 session cookie。
-->

---
class: dense
---

# 登入的資料細項

| 項目 | 內容 |
| --- | --- |
| 授權請求參數 | `response_type=code` · `client_id=pep` · `scope=openid profile` · `state` · `nonce` · `code_challenge`（S256）· `prompt=login` |
| WebAuthn 設定 | RP ID ＝ 展示網域 · User verification `required` · 不限制驗證器類型（手機跨裝置／安全金鑰皆可） |
| Token 交換 | PEP 以 HTTP Basic 帶 client secret + `code_verifier`，走容器內部網路（backchannel） |
| ID token 驗證 | JWKS 簽章 · `iss` · `aud=pep` · `nonce` 等於登入時產生的值 |
| Access token 驗證 | JWKS 簽章 · `azp=pep`（確認是發給這個 PEP 的）· `sub` 與 ID token 一致 |
| PEP 使用的 claims | `sub` · `preferred_username` · `realm_access.roles` · `amr` · `azp` · `exp` |
| Session 效期 | min（access token 效期, ID token 效期, 15 分鐘） |
| 登入 cookie | `__Host-zt_login` = state，10 分鐘；callback 時比對後立即清除 |
| Session cookie | `__Host-zt_session`：`Secure` · `HttpOnly` · `SameSite=Lax` · `Path=/` |

<div class="mt-4 text-sm opacity-80">
<code>__Host-</code> 前綴：瀏覽器只接受 HTTPS、無 Domain、Path=/ 的 cookie，子網域無法覆寫
</div>

<!--
（約 25 秒）
這頁是登入細項。重點是三個檢查：state 綁定瀏覽器防 login CSRF；PKCE 防授權碼被攔截後拿去換 token；access token 檢查 azp，確保是發給我們這個 PEP 的，不是別的 client 的 token。
session 效期取三者最短，最多 15 分鐘。cookie 都用 __Host- 前綴。
-->

---
clicks: 4
class: compact
---

# 資料交換 ②：每一個資源請求

<RequestFlow :step="$clicks" />

<!--
（約 45 秒）
登入後的每一個請求。PEP 先查 session，沒有或已過期就導向登入，連 PA 都不問。

[click] 通過後 PEP 組出 AccessContext 送 PA，PA 原封不動轉給 PE。

[click] PE 到 Redis 查威脅情資，然後依序評估 8 項條件，全部列在 checks 裡回傳。

[click] PA 依結果更新授權：允許就 SADD grants，拒絕就 SREM，並把這筆決策寫進稽核 stream，再回給 PEP。

[click] 允許時 PEP 轉發，並且刪掉 Cookie 和 Authorization，改加 X-ZT-User；拒絕就回 403 並顯示原因。PEP 不快取任何授權。
-->

---
clicks: 6
class: compact
---

# 資料交換 ③：AccessContext 與 Decision

<div grid="~ cols-[0.8fr_1.2fr] gap-4" class="mt-2 zt-code">
<div>

<div class="text-sm mb-1 opacity-80">PEP → PA → PE　<b>AccessContext</b></div>

```jsonc {all|2|5-7|8|10}
{
  "session_id": "7c1e9a0f3b2d…", // 64 hex，只在 PEP↔PA
  "subject": "Keycloak sub",
  "username": "alice",
  "roles": ["default-roles-zt"],
  "device_id": "",  // 裝置憑證 CN
  "device_fp": "",  // 憑證 DER 的 SHA-256
  "source_ip": "172.18.0.5", // TCP 對端
  "resource": "b",
  "expires_at": 1791902275 // ≤ token 效期
}
```

</div>
<div>

<div class="text-sm mb-1 opacity-80">PE → PA → PEP　<b>Decision</b>（alice 用借用電腦存取 b）</div>

```jsonc {all|2-3|10-15}
{
  "allow": false,
  "reason": "此資源需要受管理裝置，請使用已配置裝置憑證的客戶端",
  "checks": [ // 8 項全部評估；pass 的 detail 省略
    { "name": "資源政策", "result": "pass", … },
    { "name": "使用者身分", "result": "pass", … },
    { "name": "登入效期", "result": "pass", … },
    { "name": "威脅情資：來源 IP", "result": "pass", … },
    { "name": "威脅情資：帳號", "result": "pass", … },
    { "name": "裝置身分", "result": "fail",
      "detail": "此資源需要受管理裝置…" },
    { "name": "裝置狀態（CDM）", "result": "skip",
      "detail": "沒有裝置憑證" },
    { "name": "角色", "result": "fail",
      "detail": "缺少必要角色 [admin]" }
  ]
}
```

</div>
</div>

<!--
（約 40 秒）
這是 PEP、PA、PE 之間實際傳的兩個 JSON。

[click] session_id 是 64 字元的隨機值，它就是 bearer cookie，所以只在 PEP 和 PA 之間傳，稽核事件只保留前 12 碼。

[click] 裝置身分是憑證的 CN 和指紋。借用電腦沒有憑證，所以兩個都是空的，PE 就知道這是未受管理裝置。

[click] source_ip 是 PEP 實際看到的 TCP 對端，不信任 X-Forwarded-For；公開入口看到的是 Caddy 的 IP。

[click] expires_at 不超過 token 效期，PE 每次都會檢查。

[click] Decision 這邊，allow 和 reason 給 PEP 用。

[click] checks 列出全部 8 項。這個例子有兩項失敗：裝置身分和角色；reason 取第一個失敗的條件。CDM 是 skip，因為沒有憑證就沒有東西可以查。
-->

---
clicks: 4
class: compact
---

# PE 的判斷：8 項條件（§3.3.1）

<div class="text-sm opacity-70 -mt-1 mb-2">
全部評估、全部記錄；第一個失敗的條件就是拒絕原因。按空白鍵切換情境，也可以直接點
</div>

<PolicyEngine :step="$clicks" />

<!--
（約 40 秒）
PE 依序評估 8 項條件，資料來源標在第三欄。展示用的都是借用電腦，沒有裝置憑證，所以裝置狀態（CDM）這項一律略過。
第一個情境：alice 進 a，a 允許未受管理裝置，結果允許。

[click] alice 進 b：裝置身分失敗，角色也失敗，拒絕原因取第一個，也就是需要受管理裝置。

[click] bob 進 c：c 要 admin，但允許未受管理裝置，bob 是 admin，所以允許。

[click] bob 進 b：就算是 admin，借用電腦沒有裝置憑證，還是卡在裝置身分。

[click] alice 被列進風險名單：第五項威脅情資帳號失敗。任何 PIP 查詢失敗，也一律拒絕。
-->

---
clicks: 4
---

# 資料交換 ④：PIP 事件驅動的撤銷

<RevokeFlow :step="$clicks" class="mt-4" />

<div v-click="4" class="mt-3 text-sm opacity-80">
PEP <b>不快取授權</b>：撤銷通知只是加速，真正的保證是「下一個請求一定重新問 PE」。進行中的下載或長連線不會被中斷
</div>

<!--
（約 30 秒）
授權不是一次性的。管理者把 alice 加進風險名單，

[click] Redis 發出 keyspace 通知，PA 先記一筆 PIP 變動事件。

[click] PA 把所有 session 已授權的資源重新問一次 PE；另外每 30 秒也會定期重新評估。

[click] PE 拒絕，PA 移除 grant、發 pub/sub 通知 PEP，並記一筆撤銷事件，裡面含 8 項 checks。

[click] alice 重新整理就拿到 403。PEP 不快取授權，就算通知沒送到，下一個請求也會被擋。
-->

---
layout: center
class: text-center
---

# 謝謝聆聽

<div class="mt-6 font-mono opacity-70">
https://zt.cloudowo.com
</div>

<div class="mt-2 text-sm opacity-50">
參考：NIST SP 800-207 · FIDO Alliance Passkeys
</div>

<div class="mt-10 text-xs opacity-40">
附錄：教授提問 —— 私鑰被匯出怎麼辦？為什麼自己寫 PEP？
</div>

<!--
（約 5 秒）
以上，謝謝大家。後面附錄有上次會議兩個提問的回答，有問題歡迎提問。
-->

---
class: dense
---

# 附錄 A：私鑰被匯出，其他設備能登入嗎？

<div grid="~ cols-2 gap-6" class="mt-2">
<div>

**以目前實作：可以**，前提是對方也通過使用者的 passkey 驗證

<div class="text-sm">

- 裝置憑證以 p12 交付，私鑰**可匯出**
- PEP 只比對憑證指紋：複製的 p12 指紋完全相同
- CDM 以 CN 為 key：複製的裝置也被當成合規

</div>

| 攻擊者擁有 | 結果 |
| --- | --- |
| 只有 p12 | 能過 mTLS，但沒有 passkey 無法登入 |
| p12 + passkey（或本人配合） | 可以登入，裝置身分形同失效 |
| 帶憑證的 session cookie | 換電腦 → 指紋不符 → 403 |
| 未受管理裝置的 session cookie | **被偷就能用**（已知限制） |

</div>
<div>

**改進方向**

<div class="nowrap-last">

| 層級 | 做法 | 狀態 |
| --- | --- | --- |
| 防止匯出 | 私鑰在 TPM／Secure Enclave 產生，不可匯出 | 後續 |
| 證明在硬體內 | attestation：step-ca `device-attest-01`、Apple MDA、TPM | 後續 |
| 偵測複製 | 同一指紋短時間從不同來源出現 → 列入 TI 並撤銷 | 待做 |
| 縮短損害 | 裝置憑證短效期、可撤銷 | 後續 |

</div>

<div class="mt-3 text-xs opacity-75">
同步型 passkey（iCloud 鑰匙圈、Google 密碼管理工具）本來就會出現在同帳號的多台裝置，只能證明「使用者」<br>
待做實驗：把 <code>laptop1.p12</code> 複製到第二台電腦，記錄能否登入、稽核事件記錄了什麼，加上偵測規則後重做
</div>

</div>
</div>

<!--
附錄，Q&A 用。重點：只有私鑰不夠，還要通過 passkey；但同時拿到兩者時，目前分不出是哪一台。根本解法是硬體綁定加 attestation，短期可以先做複製偵測。
-->

---
class: dense
---

# 附錄 B：為什麼自己寫 PEP？

<div grid="~ cols-[1.3fr_1fr] gap-6" class="mt-2">
<div>

| 方案 | 類型 | 與本專案的差距 |
| --- | --- | --- |
| Envoy + `ext_authz` | 通用代理 + 外部授權 | 可直接接我們的 PA；但要自寫 ext_authz 服務，登入流程另外處理 |
| OPA | 政策引擎（Rego） | 只取代 PE，不是 PEP |
| Pomerium | 完整零信任存取代理 | 內建 OIDC、裝置身分；黑盒較多，不易拆開講 800-207 各角色 |
| oauth2-proxy | OIDC 登入代理 | 只做登入，沒有政策判斷與裝置狀態 |

</div>
<div class="text-sm leading-relaxed">

**選擇自寫的理由**

- 教學原型：讓 PEP / PA / PE 的**邊界與資料交換**看得見，一個角色一個服務
- 逐請求授權、撤銷、裝置綁定、fail closed 都在一千多行 Go 內，`go test -race ./...` 可完整測試

**正式環境建議**

- PEP 改用 Envoy，經 `ext_authz` 呼叫 PA；PE 可換 OPA；mTLS 與連線管理交給 Envoy
- 選做：Envoy 版 PEP 當對照組，共用同一套 PA / PE

</div>
</div>

<!--
附錄，Q&A 用。我們把教授的問題理解為「為什麼不用 Envoy 或現成 PEP」。如果教授的意思是要求改用 Envoy，就回答：架構上 PA / PE 不用改，只要把 PEP 換成 Envoy + ext_authz，這是選做的對照實驗。
-->
