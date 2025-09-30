---
theme: seriph
background: https://api.cloudowo.com/api/v1/pixiv/image/107792563
title: Desktop Pet - 智能桌面寵物
class: text-center
drawings:
    persist: false
transition: slide-left
mdc: true
colorSchema: auto
fonts:
    mono: 'Fira Code'
---

# Desktop Pet

## 智能桌面寵物 - 專題報告

<div class="pt-12">
  <span class="px-2 py-1 rounded cursor-pointer"
        hover="bg-white bg-opacity-10">
    個人助理的未來形態
  </span>
</div>





---
layout: center
class: text-center
---

# 專案概述

<div class="grid grid-cols-2 gap-12 mt-8">

<div>

## 🎯 專案目標
- 打造智能桌面夥伴
- 提供個人化助理服務
- 結合AI與可愛外觀
- 增強使用者體驗

</div>

<div>

## 🛠️ 技術棧
- **前端**: Svelte + Tauri
- **後端**: Python FastAPI
- **AI**: OpenAI GPT / 本地模型
- **桌面**: 跨平台原生應用

</div>

</div>

---
class: flex justify-center items-center
---

<div
  absolute text-6xl
  :class="$clicks < 1 ? 'text-white' : 'translate-y--18 scale-40 text-white/30'"
  transition duration-500 ease-in-out
>
  <span>又是桌面助理?</span>
</div>

<div flex flex-col items-center>
  <v-clicks>
    <div mt-4>
      <h1 flex items-center text="5xl!">
        <span class='text-white'>這不是很常見的嗎?</span>
      </h1>
    </div>
  </v-clicks>
</div>


---
class: flex justify-center items-center
---

<div class="grid grid-cols-2 gap-4 max-w-4xl mx-auto">
  <img src="/Clipboard01.png" class="rounded-lg shadow-lg hover:scale-105 transition-transform duration-300" />
  <img src="/Clipboard02.png" class="rounded-lg shadow-lg hover:scale-105 transition-transform duration-300" />
  <img src="/Clipboard03.png" class="rounded-lg shadow-lg hover:scale-105 transition-transform duration-300" />
  <img src="/Clipboard04.png" class="rounded-lg shadow-lg hover:scale-105 transition-transform duration-300" />
</div>

---
class: flex justify-center items-center
---

<div
  absolute text-6xl
  :class="$clicks < 1 ? 'text-white' : 'translate-y--18 scale-40 text-white/30'"
  transition duration-500 ease-in-out
>
  <span>那為何還要做?</span>
</div>

<div flex flex-col items-center>
  <v-clicks>
    <div mt-4>
      <h1 flex items-center text="5xl!">
        <span class='text-white'>使用別人做的老婆 不就是NTR嗎?</span>
      </h1>
    </div>
  </v-clicks>
</div>



---
layout: center
class: text-center
---

# 系統架構

```mermaid {theme: 'dark', scale: 0.8}
graph TB
    A[桌面前端] --> B[API]
    B --> C[後端服務]
    C --> D[AI 模型接口]
    C --> E[資料庫 SQLite]
    C --> F[系統API整合]

    G[用戶互動] --> A
    D --> H[OpenAI GPT]
    D --> I[本地 LLM]
    F --> J[檔案系統]
    F --> K[系統通知]


```




---
layout: center
class: space-2
---

# 開發挑戰

<div class="flex-col flex gap-4">

<v-clicks>
<div border="2 solid pink-800/50" rounded-lg>
  <div flex items-center bg="pink-800/30" px-3 py-2 text-pink-300>
    <div text-xs>
      <span>記憶系統</span>
    </div>
  </div>
  <div bg="pink-800/10" px-4 py-3>
    <div>
      <span>
        作為一個仿生智能助理，記憶系統不只是 RAG 就好
      </span>
    </div>
    <div text-xs flex gap-2 mt-1 text-zinc-400>
        <div>就像人類會在晚上整理記憶</div>
        <div>ai也需要整理記憶 刪除不需要的資料</div>
    </div>
  </div>
</div>
<div border="2 solid violet-800/50" rounded-lg>
  <div flex items-center bg="violet-800/30" px-3 py-2 text-violet-300>
    <div text-xs>
      <span>情緒模擬</span>
    </div>
  </div>
  <div bg="violet-800/10" px-4 py-3>
    <div>
      <span>
        真實的情緒反應不僅僅是預設回應，需要基於上下文和記憶進行動態情緒計算
      </span>
    </div>
    <div text-xs flex gap-2 mt-1 text-zinc-400>
      <div>情緒狀態的持續性和變化</div>
      <div>多維度情緒向量計算</div>
      <div>情緒與記憶的雙向影響</div>
    </div>
  </div>
</div>
<div border="2 solid blue-800/50" rounded-lg>
  <div flex items-center bg="blue-800/30" px-3 py-2 text-blue-300>
    <div text-xs>
      <span>資料抓取</span>
    </div>
  </div>
  <div bg="blue-800/10" px-4 py-3>
    <div>
      <span>
        桌面寵物(老婆)需要主動感知用戶的操作環境，而不是被動等待輸入
      </span>
    </div>
    <div text-xs flex gap-2 mt-1 text-zinc-400>
      <div>跨應用程式的資料收集</div>
      <div>隱私保護下的環境感知</div>
      <div>即時資料處理與分析</div>
    </div>
  </div>
</div>
</v-clicks>
</div>


---
layout: image-right
image: image.png

---

## 休比 Üc207Pr4f57t9
開發計畫

- **Alles-Lösen** 用戶資訊收集器
- **Sisters**  MAS系統



<div class="absolute bottom-4 left-8 text-sm opacity-70">
"想要了解『心』是什麼..." - 休比
</div>
---
layout: end
class: text-center
---

# 謝謝聆聽

## Desktop Pet - 智能桌面寵物


<div class="abs-br m-6 flex gap-2">
  <div class="text-sm opacity-50">
    專題報告 | 2025
  </div>
</div>

