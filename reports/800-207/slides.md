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
duration: 5min
fonts:
  sans: Noto Sans TC
  serif: Noto Serif TC
  mono: JetBrains Mono
---

# 無密碼零信任存取

## NIST SP 800-207




<div class="pt-10">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    開始 <carbon:arrow-right class="inline"/>
  </span>
</div>

<!--
（約 15 秒）
大家好，今天要分享的是我們照 NIST SP 800-207 做的零信任存取原型。
情境很簡單：你跟別人借一台電腦，什麼都不裝、也不打密碼，只用自己手機上的 passkey 登入，然後由系統決定這台電腦可以碰哪些東西。
這次也照上次會議的建議，把架構跟資料交換的細節補進來了。
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
</div>
</div>

<!--
（約 25 秒）
800-207 最核心的觀念就一句話：不會因為你在哪個網路就相信你，每一次存取都要重新檢查。

[click] 它把系統拆成兩層。使用者只能透過 PEP 碰到資源，後面由 PE 判斷、PA 負責開關。

[click] PE 判斷的時候，會參考裝置狀態、威脅情資這些資訊來源，叫做 PIP。

[click] 我們的 PEP、PA、PE 都是自己用 Go 寫的，登入交給 Keycloak，威脅情資放在 Redis，是模擬的資料。
-->

---
clicks: 4
class: compact
---

# 詳細架構圖

<ArchDetail :step="$clicks" />

<!--
（約 45 秒）
這張是實際部署的樣子。上面是對外網段，下面是內部的控制平面，右邊兩個應用各自關在獨立的區域：影音庫 Lithium 跟伺服器狀態 YASS。

[click] 入口的部分，瀏覽器走 HTTPS 進 Caddy，再轉給 PEP。借來的電腦沒有裝置憑證，所以一律當成未受管理的裝置。

[click] 控制平面就是 PEP 問 PA、PA 問 PE，授權結果跟稽核紀錄都存在 Redis。

[click] 資料平面的重點是，只有 PEP 進得去這兩個應用。所以 PEP 直接用一個 header 告訴應用「這個人是誰」，應用就不用自己做登入。

[click] 顏色代表連線的保護程度：綠色有 TLS，橘色靠 token 驗證，紅色只靠網段隔離，這是目前已知的限制。
-->


---
clicks: 4
class: compact
---

# 資料交換 ①：登入

<LoginFlow :step="$clicks" />

<!--
（約 30 秒）
先看登入。沒有 session 的話，PEP 會把你導去 Keycloak。

[click] Keycloak 只問帳號、不問密碼，直接要求 passkey。

[click] 電腦跳出 QR code，用手機掃、解鎖，手機簽名後交給 Keycloak 驗證。

[click] 接著回到 PEP，PEP 確認完成登入的是同一個瀏覽器。

[click] 最後 PEP 跟 Keycloak 換到 token、驗證過，就發 session 給你。
-->



---
clicks: 4
class: compact
---

# 資料交換 ②：每一個資源請求

<RequestFlow :step="$clicks" />

<!--
（約 25 秒）
登入之後，每一個請求都會走這條路。PEP 先檢查 session，沒有就請你重新登入。

[click] 有的話，PEP 把請求資訊交給 PA，PA 再去問 PE。

[click] PE 把 8 項條件跑一遍，把結果回傳。

[click] PA 依結果更新授權，順便記一筆稽核紀錄。

[click] 允許就轉給應用，不允許就回 403。而且 PEP 不記結果，每次都重新問。
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
  "resource": "status",
  "expires_at": 1791902275 // ≤ token 效期
}
```

</div>
<div>

<div class="text-sm mb-1 opacity-80">PE → PA → PEP　<b>Decision</b>（alice 用借用電腦存取 status）</div>

```jsonc {all|2-3|10-15}
{
  "allow": false,
  "reason": "缺少必要角色 [admin]",
  "checks": [ // 8 項全部評估；部分 detail 省略
    { "name": "資源政策", "result": "pass", … },
    { "name": "使用者身分", "result": "pass", … },
    { "name": "登入效期", "result": "pass", … },
    { "name": "威脅情資：來源 IP", "result": "pass", … },
    { "name": "威脅情資：帳號", "result": "pass", … },
    { "name": "裝置身分", "result": "pass",
      "detail": "未受管理裝置；政策允許此資源" },
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
（約 25 秒）
這頁是 PEP、PA、PE 之間實際傳的資料。左邊是 PEP 送出去的請求資訊。

[click] 裡面有 session，只在內部傳。

[click] 有使用者角色跟裝置資訊，借來的電腦裝置這兩格是空的。

[click] 有來源 IP，

[click] 還有過期時間。

[click] 右邊是 PE 的回覆：能不能過、為什麼。

[click] 8 項結果都會列出來，這個例子是卡在角色，因為 alice 不是 admin。
-->

---
clicks: 3
class: compact
---

# PE 的判斷：8 項條件（§3.3.1）

<div class="text-sm opacity-70 -mt-1 mb-2">
全部評估、全部記錄；第一個失敗的條件就是拒絕原因。按空白鍵切換情境，也可以直接點
</div>

<PolicyEngine :step="$clicks" />

<!--
（約 30 秒）
這頁把 8 項條件跟每一項的資料來源都列出來了，我們直接看幾個情境。
第一個，alice 開影音庫，只要有登入就可以，通過。

[click] alice 改開伺服器狀態，這個要 admin，卡在角色這關，403。

[click] 換成有 admin 的 bob，用同一台電腦開就過了，差別就只在角色。

[click] 最後把 alice 放進風險名單，威脅情資那關直接擋下來，連影音庫都進不去。另外只要任何一個資料來源查不到，也是一律拒絕。
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
（約 25 秒）
最後是撤銷。假設管理者把 alice 加進風險名單，

[click] Redis 會馬上通知 PA。

[click] PA 把已經給出去的授權再拿去問 PE 一次。

[click] PE 說不行，PA 就收回授權、通知 PEP。

[click] alice 下一個請求就是 403。就算通知沒送到也沒關係，PEP 本來就每次都重新問；不過已經在下載的東西不會被中斷。
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
以上就是我們的報告，謝謝大家。後面附錄是上次兩個提問的回答，有問題歡迎再問。
-->
