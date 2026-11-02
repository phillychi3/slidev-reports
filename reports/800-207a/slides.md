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
大家好，這次是零信任存取原型的第二版。情境不變：借一台別人的電腦，不裝東西、不輸入密碼，用手機上的 passkey 登入。
差別是這次改依 NIST SP 800-207A，平台也從單台主機的 docker-compose 換成 GCP 上的 Kubernetes 加 service mesh。
-->

---
class: dense
---

# 800-207A 要求對應

<div class="text-sm opacity-70 -mt-1 mb-3">
政策都在 <code>deploy/k8s/manifests/</code>：<code>50-identity-tier.yaml</code>（身分層）、<code>60-network-tier.yaml</code>（網路層）
</div>

| 800-207A 要求 | 實作 | 狀態 |
| --- | --- | --- |
| 服務間連線加密 | `PeerAuthentication` STRICT：四個 namespace 只收 mTLS | <span class="st done">完成</span> |
| 服務身分驗證 | 每個 workload 一個 ServiceAccount → `spiffe://<專案>.svc.id.goog/ns/<ns>/sa/<sa>`，Mesh CA 簽發、自動輪替 | <span class="st done">完成</span> |
| 服務授權 | `AuthorizationPolicy`：每個 namespace 預設 `allow-nothing`，逐條允許「來源身分 → 目的、方法、路徑」 | <span class="st done">完成</span> |
| 終端使用者驗證 | passkey＋OIDC（PEP 驗 token）；PEP 對每個請求簽發 `X-ZT-Assertion`，資源端 sidecar 以 `RequestAuthentication` 驗證 | <span class="st done">完成</span> |
| 終端使用者授權 | PE 逐請求判斷；資源端 sidecar 再檢查聲明（YASS 需要 `roles` 含 `admin`） | <span class="st done">完成</span> |
| 網路層政策 | `NetworkPolicy`（GKE Dataplane V2），預設進出皆拒 | <span class="st done">完成</span> |
| 出站限制 | `Sidecar` `REGISTRY_ONLY`：應用只能連到列出的服務 | <span class="st done">完成</span> |
| 裝置身分安全地傳給 PEP | gateway TLS MUTUAL → XFCC；PEP 只在上一跳是 gateway 的 SPIFFE ID 時採信 | <span class="st done">完成</span> |
| 政策經 CI/CD 部署 | Terraform（GCP）＋ manifest 進版控；目前手動 `terraform apply`／`deploy.sh` | <span class="st part">部分</span> |
| 多雲／多叢集 | 只有單一 GKE 叢集 | <span class="st todo">未做</span> |

<!--
（約 30 秒）
這張是 800-207A 的要求和我們的對應。前八項都已經在 GKE 上部署並驗證：服務間加密、服務身分、服務授權、使用者驗證與授權、網路層、出站限制，還有裝置身分怎麼安全地從邊界傳到 PEP。
沒做完的是兩項：政策目前進版控但還是手動套用，沒有接 CI/CD；800-207A 強調的多雲、多地點，我們只有一個叢集。
-->

---
clicks: 4
class: compact
---

# 詳細架構圖（GKE + Cloud Service Mesh）

<MeshArch :step="$clicks" />

<!--
（約 60 秒）
這是 GKE 上的實際部署。左邊是客戶端，中間 zt-gateway 是唯一對外的入口，zt-system 是控制平面，右邊兩個應用各自在自己的 namespace。每個 workload 有自己的 ServiceAccount，就是它的 SPIFFE 身分，憑證由 Google 代管的 Mesh CA 簽發、自動輪替。

[click] 入口：GCP 的 L4 load balancer 後面是 Istio ingress gateway。公開網域不收用戶端憑證，所以借用電腦一律是未受管理裝置；受管理電腦走 device 子網域，gateway 用 TLS MUTUAL 驗證裝置憑證，再把憑證資訊放在 XFCC header 交給 PEP。/auth 直接轉給 Keycloak。

[click] 控制平面：流程跟上一版一樣，PEP 問 PA，PA 問 PE，PE 查 Redis。差別是每一跳都經過 sidecar 的 mTLS，而且有一條 AuthorizationPolicy 規定誰能打誰的哪個路徑，例如只有 pa 能 POST /decide。

[click] 資料平面：PEP 轉發時簽一張使用者聲明 X-ZT-Assertion，資源前面的 sidecar 驗證來源是 PEP 的身分、而且聲明有效。lithium 連不到 yass：身分層的出站限制和網路層都會擋。

[click] 依保護等級上色：每一條服務間連線都是雙向驗證的 mTLS，藍色的幾條還多了使用者聲明。唯一的例外是申請 Let's Encrypt 憑證時的 ACME 驗證，圖上沒畫。
-->

---
clicks: 4
class: compact
---

# 資料交換 ①：每一個資源請求

<RequestFlow :step="$clicks" />

<!--
（約 45 秒）
以 Lithium 的一個 API 請求為例。瀏覽器經 gateway 到 PEP，gateway 到 PEP 這段已經是 mTLS。PEP 先看 XFCC 最後一段是不是 gateway 的身分，再查 session、比對裝置指紋。

[click] PEP 送 AccessContext 給 PA，PA 轉給 PE。這兩跳 sidecar 都會驗證對方身分，AuthorizationPolicy 只允許 pep 打 /access、只允許 pa 打 /decide。

[click] PE 的判斷完全沿用上一版：查威脅情資、評估 8 項條件，PA 記錄 grants 和稽核事件。

[click] 拒絕就回 403。允許時是這次新增的部分：PEP 簽一張使用者聲明，ES256、audience 是 media、60 秒，跟 X-ZT-User 一起轉發，並刪掉 Cookie、Authorization 和 XFCC。

[click] 資源前的 sidecar 驗證來源是 sa/pep，聲明的簽章、issuer、audience、效期都有效才放行。nginx 再轉給後端時，後端的 sidecar 會再驗一次。
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
（約 40 秒）
左邊是 PEP 簽的聲明，右邊是 YASS 這個 namespace 的 sidecar 政策。

[click] audience 綁定資源名稱，所以拿 media 的聲明去打 YASS 會被拒絕。

[click] roles 從 Keycloak token 帶過來，資源端可以自己再檢查一次。

[click] 效期 60 秒，而且不超過登入效期；jti 每個請求不同。

[click] 右邊第一段是 RequestAuthentication：issuer 必須是 zt-pep、audience 必須是 status，用內嵌的公鑰驗簽章，而且只看 x-zt-assertion 這個 header。

[click] 第二段是授權：上一跳的 mTLS 身分必須是 PEP 的 ServiceAccount，

[click] 而且要帶一張驗證通過的聲明，

[click] 聲明裡的 roles 還要有 admin。這等於 PE 判斷完之後，資源端再獨立判斷一次；就算 PE 被誤設成允許，沒有 admin 還是進不去。
-->

---
clicks: 3
class: compact
---

# 資料交換 ③：裝置身分經 gateway 傳給 PEP

<DeviceFlow :step="$clicks" class="mt-2" />

<!--
（約 30 秒）
裝置身分還是 X.509 憑證，但 TLS 現在在 gateway 終結，PEP 拿不到原始憑證。受管理電腦連 device 子網域，gateway 用裝置 CA 驗證憑證，沒有憑證握手就失敗。

[click] gateway 設成 SANITIZE_SET：先丟掉用戶端自己送來的 XFCC，只寫入它驗證過的憑證 Hash 和 Subject，再用 mTLS 送給 pep-managed。

[click] PEP 這邊的 sidecar 會在 XFCC 最後附上上一跳的 SPIFFE ID。PEP 只在最後一段是 gateway 時，才採信前面的裝置資訊和來源 IP；這段邏輯在 mesh.go 的 meshIdentity。

[click] 之後跟上一版一樣，PE 用裝置名稱查 CDM 狀態。實測時，pep-managed 從 XFCC 拿到的指紋和本機 openssl 算出來的一致；公開入口帶假的 XFCC 仍然是未受管理裝置。
-->

---
clicks: 5
class: compact
---

# 兩層政策實測：誰擋下了請求

<div class="text-sm opacity-70 -mt-1 mb-3">
2026-10-30 在 GKE 上從測試 pod 實際發出；按空白鍵切換情境，也可以直接點
</div>

<TierProbe :step="$clicks" />

<!--
（約 60 秒）
這些是在叢集裡實際打出來的結果。第一個是正常路徑：PEP 的身分加上 media 聲明，兩層都放行，200。

[click] 假設 Lithium 的 nginx 被攻陷，它偽造 X-ZT-User: bob 去打後端。網路層放行，因為 nginx 本來就該連後端；但身分層要求一張 PEP 簽的聲明，沒有就 403。如果資源只看 X-ZT-User，這一步就會成功。

[click] 測試 pod 用 default 身分，但貼上 app=pe 的標籤去連 Redis。網路層是看標籤的，被騙過了；身分層看的是 SPIFFE 身分，連線被關閉。

[click] PEP 身分加上 status 聲明，但 roles 沒有 admin：身分層的 claims 條件擋下。這代表 PE 就算誤判，資源端仍然擋得住。

[click] 沒有 sidecar 的 pod 想直接連後端：封包根本到不了 sidecar，網路層就丟掉了，逾時。這是身分層被繞過時，網路層存在的理由。

[click] 最後是橫向移動：從 lithium 連 YASS，身分層的出站限制和網路層各自都會擋。
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
以上，謝謝大家，有問題歡迎提問。
-->
