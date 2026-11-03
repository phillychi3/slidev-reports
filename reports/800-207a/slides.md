---
theme: seriph
background: https://cover.sli.dev
title: 無密碼零信任存取（800-207A）
info: |
  ## 無密碼零信任存取（800-207A）
  依 NIST SP 800-207A 改寫的教學原型：部署在 GKE + Cloud Service Mesh，服務間 mTLS 與 SPIFFE 身分、身分層＋網路層兩層政策，使用者仍以 Keycloak passkey 登入。
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

## NIST SP 800-207A

<div class="mt-4 opacity-70">
GKE · Cloud Service Mesh · passkey
</div>

<div class="pt-10">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    開始 <carbon:arrow-right class="inline"/>
  </span>
</div>

<!--
（約 15 秒）
大家好，這次要報告的是我們零信任原型的第二版。情境一樣：借一台別人的電腦，不用裝東西、也不用打密碼，拿手機的 passkey 就能登入。
這版改照 NIST 800-207A 來做，平台也從一台主機跑 docker-compose，搬到 GCP 的 Kubernetes，再加上 service mesh。
-->



---
clicks: 4
class: compact
---

# 詳細架構圖（GKE + Cloud Service Mesh）

<MeshArch :step="$clicks" />

<!--
（約 60 秒）
先看整個架構。左邊是使用者，中間的 zt-gateway 是唯一對外的大門，zt-system 放控制平面，右邊兩個應用各自住在自己的 namespace。
重點是每個服務旁邊都掛了一個 Envoy sidecar，就是那條綠條。服務本身只講 HTTP，加密、驗身分、授權都交給 sidecar。每個服務的 ServiceAccount 就是它的身分證，憑證由 Google 簽、自動換。

[click] 入口這邊：一般網域不收憑證，借來的電腦一律當成「沒受管理」；公司配的電腦走 device 子網域，要先拿出裝置憑證才進得來。

[click] 控制平面的流程跟上一版一樣，PEP 問 PA、PA 再問 PE。差別是現在每一跳都是 mTLS，而且有規則管「誰能打誰」，像是只有 PA 能叫 PE 做決策。

[click] 資料平面這邊，PEP 放行的時候會簽一張「使用者聲明」帶過去，資源前面的 sidecar 會檢查這張聲明。

[click] 簡單講，圖上每一條線都是雙向驗證的 mTLS，藍色那幾條還多帶了一張使用者聲明。
-->

---
clicks: 4
class: compact
---

# 資料交換 ①：每一個資源請求

<RequestFlow :step="$clicks" />

<!--
（約 30 秒）
這頁是一個請求從進來到拿到資料的過程。瀏覽器的請求經過 gateway 到 PEP，PEP 先確認使用者和裝置。

[click] 接著 PEP 去問 PA，PA 再問 PE：這個人現在能不能存取。

[click] PE 做判斷，PA 把結果記下來。

[click] 不行就回 403；可以的話，PEP 簽一張使用者聲明，跟著請求一起轉給資源。

[click] 資源那邊先驗這張聲明，沒問題才回資料。
-->

---
clicks: 7
class: compact
---

# 資料交換 ②：使用者聲明與資源端驗證

<div grid="~ cols-[0.85fr_1.15fr] gap-4" class="mt-2 zt-code">
<div>

<div class="text-sm mb-1 opacity-80">PEP → 資源　<b>X-ZT-Assertion</b>（JWT payload）</div>

```jsonc {all|3|6|9-10}
{
  "iss": "zt-pep",
  "aud": "status",       // 綁定資源；拿去打 media 無效
  "sub": "Keycloak sub",
  "preferred_username": "bob",
  "roles": ["admin", "default-roles-zt"],
  "device": "",          // 裝置 CN；未受管理為空
  "sid": "7c1e9a0f12ab", // session 前綴，只供稽核
  "iat": 1791901375,
  "exp": 1791901435,     // 60 秒，且 ≤ 登入效期
  "jti": "每個請求不同"
}
```

<div class="text-xs opacity-60 mt-1">ES256，PEP 私鑰簽章；公鑰以 JWKS 內嵌在政策裡</div>

</div>
<div>

<div class="text-sm mb-1 opacity-80">zt-status 的 sidecar　<b>50-identity-tier.yaml</b></div>

```yaml {all|5-8|16|17|18-20}
# 驗證聲明：簽章、issuer、audience、效期
kind: RequestAuthentication
spec:
  jwtRules:
    - issuer: zt-pep
      audiences: [status]
      jwks: '<PEP 公鑰 JWKS>'
      fromHeaders: [{ name: x-zt-assertion }]
---
kind: AuthorizationPolicy   # yass
spec:
  action: ALLOW
  rules:
    - from:
        - source:
            principals: [".../ns/zt-system/sa/pep"]
            requestPrincipals: ["zt-pep/*"]
      when:
        - key: request.auth.claims[roles]
          values: [admin]
```

</div>
</div>

<!--
（約 30 秒）
這頁看那張聲明長什麼樣子，還有資源端怎麼檢查它。

[click] 它寫了這張聲明是給哪個資源用的，

[click] 使用者有哪些角色，

[click] 還有很短的有效期，只有 60 秒。

[click] 資源端先確認這張聲明真的是 PEP 簽的，

[click] 再確認請求是從 PEP 來的，

[click] 有帶聲明，

[click] 而且角色要是 admin 才放行。也就是說，資源自己會再把關一次。
-->

---
clicks: 3
class: compact
---

# 資料交換 ③：裝置身分經 gateway 傳給 PEP

<DeviceFlow :step="$clicks" class="mt-2" />

<!--
（約 25 秒）
這頁是公司電腦的裝置身分怎麼傳到 PEP。電腦連線時要出示裝置憑證，由 gateway 負責驗證。

[click] 驗過之後，gateway 把憑證資訊寫進 header 傳給 PEP，使用者自己塞的會先被丟掉。

[click] PEP 確認這個 header 真的是 gateway 給的，再從裡面取出裝置名稱。

[click] 最後 PE 用裝置名稱查這台電腦的狀態，決定要不要放行。
-->

---
clicks: 5
class: compact
---

# 兩層政策實測：誰擋下了請求

<div class="text-sm opacity-70 -mt-1 mb-3">
2026-10-30 在 GKE 上從測試 pod 實際發出
</div>

<TierProbe :step="$clicks" />

<!--
（約 60 秒）
最後是實測，都是在叢集裡真的打出來的。第一個是正常情況，兩層都放行，回 200。

[click] 假設 Lithium 的 nginx 被攻破了，它假裝自己是 bob 去打後端。網路層會放行，因為 nginx 本來就能連後端；但身分層要 PEP 簽的聲明，拿不出來就 403。

[click] 再來，測試 pod 偷貼上 PE 的標籤去連 Redis。網路層只看標籤，所以被騙過了；但身分層看的是真正的身分，連線直接被切掉。

[click] 這個是帶了 PEP 的身分跟聲明，但角色不是 admin，身分層一樣擋下來。

[click] 那如果有 pod 根本沒裝 sidecar 呢？身分層管不到它，就換網路層把封包丟掉，這就是兩層都要有的原因。

[click] 最後是橫向移動：從 lithium 想連 YASS，兩層各自都會擋。
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
參考：NIST SP 800-207A · NIST SP 800-207 · Cloud Service Mesh · Istio Security
</div>

<!--
（約 5 秒）
以上就是這次的報告，謝謝大家，有問題歡迎提問。
-->
