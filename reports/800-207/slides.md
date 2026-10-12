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
duration: 4min
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
Go · Keycloak · Caddy · Redis · step-ca · Docker Compose
</div>

<div class="pt-10">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    開始 <carbon:arrow-right class="inline"/>
  </span>
</div>

<!--
（約 15 秒）
大家好，今天介紹的是一個依照 NIST SP 800-207 做的零信任存取原型。
情境很簡單：借一台別人的電腦，不裝任何東西、不輸入任何密碼，只用自己手機上的 passkey 登入，然後系統要決定這台電腦能碰哪些資源。
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
---

# 系統架構

<ArchDiagram :step="$clicks" class="mt-2" />

<!--
（約 40 秒）
這是實際部署。對外只開 Caddy 的 443，PEP 是唯一能通往資源的入口，控制平面跟兩個資源都在 internal network 裡。

[click] 登入時，瀏覽器經 Caddy 到 Keycloak，手機透過 QR code 加藍牙完成 passkey 驗證。

[click] 登入後，每一個資源請求都會由 PEP 帶著 AccessContext 問 PA，PA 再問 PE，PE 查 Redis 的裝置狀態與威脅情資。

[click] 通過才轉發。借用電腦可以進一般資源 A，但敏感資源 B 會被擋下。

[click] 最後是微分段：res-a 跟 res-b 在不同網路，就算 A 被攻陷，也連不到 B。
-->

---
clicks: 2
---

# 為什麼要無密碼？

<div grid="~ cols-[1fr_0.9fr] gap-6" class="mt-4 items-center">
<div class="text-sm">

|  | 密碼 | Passkey |
| --- | --- | --- |
| 伺服器存 | 密碼雜湊，外洩可離線破解 | **公鑰**，外洩也登入不了 |
| 釣魚網站 | 使用者照樣輸入 | 簽章綁定網域（RP ID） |
| 借用電腦 | 可能被側錄、被記住 | **不輸入任何秘密** |
| 使用者 | 記住密碼 | 手機指紋／臉部解鎖 |

</div>
<div>
  <KeyPair :step="$clicks" />
</div>
</div>

<div v-click="2" class="mt-2 px-4 py-2 rounded-lg border-l-4 border-amber-400 bg-amber-400/10 text-[0.95rem]">
⚠️ passkey 證明的是「<b>人</b>」，不是「<b>電腦</b>」→ 借用電腦一律視為<b>未受管理裝置</b>
</div>

<!--
（約 25 秒）
為什麼要無密碼？passkey 是一組公私鑰，私鑰在手機的安全晶片裡，伺服器只存公鑰，被偷了也沒用。

[click] 而且簽章綁定網域，釣魚網站就算騙使用者點進去，手機也不會替它簽。

[click] 但要特別注意：passkey 證明的是「這個人是 alice」，不代表這台借來的電腦是安全的。所以系統會把它當成未受管理裝置，這個區分後面會用到。
-->

---
clicks: 4
---

# 無密碼登入流程

<LoginFlow :step="$clicks" />

<!--
（約 40 秒）
登入流程是 OIDC 授權碼流程加上 WebAuthn。第一步，PEP 把瀏覽器導到 Keycloak，帶上 state、PKCE 跟 nonce，並要求每次都重新驗證。

[click] Keycloak 只問帳號，沒有密碼欄位，接著發出 WebAuthn challenge，綁定網域 zt.cloudowo.com。

[click] 借用電腦顯示 QR code，用手機掃描、藍牙確認在附近，手機用指紋或臉部解鎖後，拿私鑰簽 challenge。

[click] Keycloak 用註冊時存的公鑰驗章，通過才發授權碼。

[click] 最後 PEP 確認 state 跟 login cookie 一致，從後端通道用 code 加 PKCE verifier 換 token，驗完 nonce 跟 azp，建立最多 15 分鐘的 session。
-->

---
clicks: 5
---

# 每一個請求都重新授權

<div class="text-sm opacity-70 -mt-2 mb-4">
PE 的 criteria-based trust algorithm（800-207 §3.3.1）：依序檢查，任一項不符就拒絕
</div>

<PolicyEngine :step="$clicks" />

<!--
（約 40 秒）
登入只是拿到身分。之後每一個請求，PE 都會依序檢查四件事：身分與效期、威脅情資、裝置、資源角色。
第一個情境：alice 用借用電腦進 A，A 允許未受管理裝置，通過。

[click] 換 bob 進 B，就算 bob 是 admin，借用電腦沒有裝置憑證，第三關就被擋。

[click] 如果 alice 被列進風險名單，第二關就拒絕。

[click] 換成有 mTLS 裝置憑證、而且合規的電腦，alice 進 B 還是不行，因為她不是 admin。

[click] bob 用合規裝置，全部通過。

[click] 但只要裝置狀態變成不合規，連 A 都進不去。另外，任何查詢失敗都會 fail closed。
-->

---
clicks: 4
---

# 持續評估：風險出現就撤銷

<RevokeFlow :step="$clicks" class="mt-4" />

<div v-click="4" class="mt-3 text-sm opacity-80">
PEP <b>不快取授權</b>：撤銷通知只是加速，真正的保證是「下一個請求一定重新問 PE」
</div>

<!--
（約 25 秒）
授權不是一次性的。管理者把 alice 加進風險名單，

[click] Redis 發出 keyspace 通知，

[click] PA 收到後立刻把所有 session 已授權的資源重新問一次 PE，此外每 30 秒也會定期重新評估。

[click] PE 拒絕，PA 撤銷授權並通知 PEP。

[click] alice 重新整理頁面就拿到 403。重點是 PEP 不快取授權，就算通知沒送到，下一個請求還是會被擋。
-->

---

# 總結

<div grid="~ cols-2 gap-8" class="mt-6">
<div>

### ✅ 做到了

<v-clicks>

- **全程無密碼**：Keycloak WebAuthn + 手機 passkey
- **PEP 唯一入口**：每個請求經 PA → PE，fail closed
- **裝置分級**：借用電腦只能碰一般資源
- **即時撤銷 + 微分段**：限制橫向移動

</v-clicks>

</div>
<div v-click>

### ⚠️ 誠實的限制

- CDM 裝置狀態、威脅情資為 Redis **模擬資料**
- 內部服務之間尚未全面 TLS／服務身分驗證
- 未受管理裝置的 session 是 bearer cookie
- 撤銷擋得住下一個請求，擋不住已開始的下載

</div>
</div>

<div class="mt-14 text-center text-2xl">
用 <b class="text-emerald-400">passkey</b> 證明「人」，用 <b class="text-sky-400">政策</b> 決定「每一個請求」
</div>

<!--
（約 20 秒）
總結一下：[click] 全程沒有密碼，[click] PEP 是唯一入口而且每個請求都重新授權，[click] 借用電腦只能碰一般資源，[click] 風險出現能即時撤銷，網路也有微分段。

[click] 當然這是教學原型，裝置狀態跟威脅情資是模擬的，內部通訊也還沒全面加密，這些是下一步要補的。
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

<!--
（約 5 秒）
以上，謝謝大家，有問題歡迎提問。
-->
