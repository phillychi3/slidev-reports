---
theme: default
background: "#121212"
class: "text-center"
highlighter: shiki
lineNumbers: false
drawings:
  persist: false
transition: slide-left
title: SHARE-CAT - 現代化的線上解題系統
mdc: true
---

# SHARE-CAT

<div class="text-xl text-gray-400 mt-4 animate-fade-in">
現代化的線上解題系統
</div>


<style>
h1 {
  font-size: 5rem !important;
  font-weight: 600;
  letter-spacing: 2px;
}

</style>

---

# 目錄

<div class="grid grid-cols-1 gap-6 mt-10 animate-list">
  <div v-click class="text-2xl">1. 專案背景與動機</div>
  <div v-click class="text-2xl">2. 系統功能介紹</div>
  <div v-click class="text-2xl">3. 技術架構</div>
  <div v-click class="text-2xl">4. AI 整合功能</div>
  <div v-click class="text-2xl">5. 未來發展方向</div>
</div>

<style>
h1 {
  font-size: 4rem !important;
  font-weight: 600;
}

.animate-list > div {
  transition: all 0.5s ease;
}
</style>

---

# 專案背景與動機

<div v-click class="text-2xl mb-8">
現代程式教育面臨的挑戰：
</div>

<div class="grid grid-cols-2 gap-6">
  <div v-click class="p-4 border border-blue-500 rounded-lg">
    <div class="text-xl font-bold text-blue-400 mb-2">學習資源分散</div>
    <div class="text-lg">學生需要在不同平台間切換，降低學習效率</div>
  </div>

  <div v-click class="p-4 border border-green-500 rounded-lg">
    <div class="text-xl font-bold text-green-400 mb-2">缺乏即時反饋</div>
    <div class="text-lg">傳統平台難以提供個性化的程式學習指導</div>
  </div>

  <div v-click class="p-4 border border-purple-500 rounded-lg">
    <div class="text-xl font-bold text-purple-400 mb-2">教師負擔重</div>
    <div class="text-lg">題目設計與評分耗時</div>
  </div>

  <div v-click class="p-4 border border-yellow-500 rounded-lg">
    <div class="text-xl font-bold text-yellow-400 mb-2">技術快速發展</div>
    <div class="text-lg">現有平台難以完成特殊題型</div>
  </div>
</div>

<div v-click class="mt-8 text-xl">
SHARE-CAT 致力於解決這些問題，打造一站式的現代化程式學習平台。
</div>

<style>
h1 {
  font-size: 2.5rem !important;
  font-weight: 600;
  margin-bottom: 2rem;
}
</style>

---

# 什麼是 Share-cat？

<div v-click class="text-2xl mb-6">
Share-cat 是 <span class="text-blue-400 font-bold">PLWeb 的升級版</span>，是一個整合多功能的 Online Judge 系統。
</div>

<div class="grid grid-cols-2 gap-x-8 gap-y-6 mt-6">
  <div v-click>
    <div class="text-2xl font-bold mb-4 text-gradient">四大核心功能</div>
    <div class="text-xl space-y-4">
      <div class="flex items-center"><div class="text-2xl mr-3">🏆</div> <div>競程平台 - 支援各類競賽模式與即時排名</div></div>
      <div class="flex items-center"><div class="text-2xl mr-3">📝</div> <div>考試平台 - 自動評分與防作弊機制</div></div>
      <div class="flex items-center"><div class="text-2xl mr-3">🚩</div> <div>CTF 平台 - 資安技能與實戰演練</div></div>
      <div class="flex items-center"><div class="text-2xl mr-3">📚</div> <div>學習平台 - 互動式教學與進度追蹤</div></div>
    </div>
  </div>
  <div>
    <div v-click class="text-2xl font-bold mb-4 text-gradient">與 PLWeb 的差異</div>
    <div v-click class="text-xl space-y-3">
      <div class="flex items-start">
        <div class="text-green-400 text-xl mr-2 mt-1">✓</div>
        <div>現代化 UI/UX 設計，更直覺的操作體驗</div>
      </div>
      <div class="flex items-start">
        <div class="text-green-400 text-xl mr-2 mt-1">✓</div>
        <div>更強大的程式語言支援與評測系統</div>
      </div>
      <div class="flex items-start">
        <div class="text-green-400 text-xl mr-2 mt-1">✓</div>
        <div>整合 AI 輔助功能，提高教學與學習效率</div>
      </div>
      <div class="flex items-start">
        <div class="text-green-400 text-xl mr-2 mt-1">✓</div>
        <div>最新技術崁，擴展性更佳，支援大規模用戶</div>
      </div>
    </div>
  </div>
</div>

<style>
h1 {
  font-size: 2.5rem !important;
  font-weight: 600;
  margin-bottom: 1.5rem;
}

.text-gradient {
  background: linear-gradient(90deg, #4299e1, #9f7aea);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}
</style>

---

# Share-cat 可以做到什麼？

<div class="grid grid-cols-2 gap-x-8 gap-y-6 mt-6">
  <div v-click>
    <div class="text-2xl font-bold mb-4 text-blue-400">教師視角</div>
    <div class="space-y-4 text-xl">
      <div class="flex items-start">
        <div class="text-2xl mr-3">📝</div>
        <div>輕鬆建立測驗與競賽，自動評分</div>
      </div>
      <div class="flex items-start">
        <div class="text-2xl mr-3">📈</div>
        <div>分析學生表現數據，掌握學習狀況</div>
      </div>
      <div class="flex items-start">
        <div class="text-2xl mr-3">🤖</div>
        <div>利用 AI 協助生成題目與測試案例</div>
      </div>
    </div>
  </div>
    <div v-click>
    <div class="text-2xl font-bold mb-4 text-green-400">學生視角</div>
    <div class="space-y-4 text-xl">
      <div class="flex items-start">
        <div class="text-2xl mr-3">💻</div>
        <div>參與各種程式設計競賽與挑戰</div>
      </div>
      <div class="flex items-start">
        <div class="text-2xl mr-3">🧩</div>
        <div>通過AI互動幫助釐清問題</div>
      </div>
      <div class="flex items-start">
        <div class="text-2xl mr-3">📊</div>
        <div>追蹤個人學習進度與能力成長</div>
      </div>
      <div class="flex items-start">
        <div class="text-2xl mr-3">👥</div>
        <div>與同儕交流解題思路，共同成長</div>
      </div>
    </div>
  </div>
</div>

<div v-click class="mt-8 p-4 border border-yellow-400 rounded-lg">
  <div class="text-2xl font-bold text-yellow-400 mb-2">未來展望</div>
  <div class="text-xl">
    支援 AI 模型競賽、程式碼智能分析、學生論壇等
  </div>
</div>

<style>
h1 {
  font-size: 2.5rem !important;
  font-weight: 600;
  margin-bottom: 1.5rem;
}
</style>

---

# Share-cat 預計可以支援哪些語言？

<div class="grid grid-cols-3 gap-12 mt-8">
  <div v-click class="language-card">
    <div class="text-6xl"><logos-python /></div>
    <div class="text-2xl mt-4">Python 3</div>
  </div>
  <div v-click class="language-card">
    <div class="text-6xl"><logos-java /></div>
    <div class="text-2xl mt-4">Java</div>
  </div>
  <div v-click class="language-card">
    <div class="text-6xl"><logos-php /></div>
    <div class="text-2xl mt-4">PHP</div>
  </div>
</div>

<div class="grid grid-cols-4 gap-8 mt-12">
  <div v-click class="language-card-sm">
    <div class="text-4xl"><logos-c /></div>
    <div class="text-xl mt-3">C</div>
  </div>
  <div v-click class="language-card-sm">
    <div class="text-4xl"><logos-c-plusplus /></div>
    <div class="text-xl mt-3">C++</div>
  </div>
  <div v-click class="language-card-sm">
    <div class="text-4xl">
      <logos-javascript />
    </div>
    <div class="text-xl mt-3">JavaScript</div>
  </div>
  <div v-click class="language-card-sm">
    <div class="text-4xl">
      <logos-rust />
    </div>
    <div class="text-xl mt-3">Rust</div>
  </div>
</div>
<div v-click>
more...
</div>

<style>
h1 {
  font-size: 2.5rem !important;
  font-weight: 600;
  margin-bottom: 2rem;
}

.language-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 1.5rem;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.05);
  transition: all 0.3s ease;
}

.language-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
  background: rgba(255, 255, 255, 0.1);
}

.language-card-sm {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 1rem;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.03);
  transition: all 0.3s ease;
}

.language-card-sm:hover {
  transform: translateY(-3px);
  background: rgba(255, 255, 255, 0.08);
}
</style>

---

# AI 功能

<div class="grid grid-cols-2 gap-x-10 mt-8">
  <div>
    <div v-click class="text-2xl font-bold mb-6 text-blue-400">教師端 AI 功能</div>
    <div v-click class="p-4 border border-blue-400 rounded-lg mb-2">
      <div class="text-xl font-bold mb-2">🤖 題目生成</div>
      <div class="text-lg">根據課程要求，自動生成符合教學目標的題目</div>
    </div>
    <div v-click class="p-4 border border-blue-400 rounded-lg mb-2">
      <div class="text-xl font-bold mb-2">📊 學習數據分析</div>
      <div class="text-lg">分析學生解題模式與錯誤類型</div>
    </div>
    <div v-click class="p-4 border border-blue-400 rounded-lg">
      <div class="text-xl font-bold mb-2">🧪 測試案例生成</div>
      <div class="text-lg">自動生成測試案例，確保題目評測的準確性</div>
    </div>
  </div>
  <div>
    <div v-click class="text-2xl font-bold mb-6 text-green-400">學生端 AI 功能</div>
    <div v-click class="p-4 border border-green-400 rounded-lg mb-2">
      <div class="text-xl font-bold mb-2">💡 智能提示</div>
      <div class="text-lg">提供針對性的解題提示，不直接給出答案，而是引導思考方向</div>
    </div>
    <div v-click class="p-4 border border-green-400 rounded-lg mb-2">
      <div class="text-xl font-bold mb-2">💬 程式碼對話助手</div>
      <div class="text-lg">回答關於程式概念的問題，解釋錯誤原因，建議優化方向</div>
    </div>
  </div>
</div>

<div v-click class="mt-2 p-4 bg-gradient-to-r from-blue-900 to-purple-900 rounded-lg text-xl">
AI 功能設計原則：輔助而非取代學習過程，強調培養解題思路與程式設計能力
</div>

<style>
h1 {
  font-size: 2.5rem !important;
  font-weight: 600;
  margin-bottom: 1.5rem;
}
</style>

---

# 專案架構

<div class="grid grid-cols-2 gap-10 mt-6">
  <div>
    <div v-click class="p-5 border border-pink-500 rounded-lg relative overflow-hidden mb-8">
      <div class="text-2xl font-bold text-pink-400 mb-4">前端 - Nekopara</div>
      <div class="text-lg mb-3">• React & Next.js </div>
      <div class="text-lg mb-3">• 現代化的UI介面</div>
      <div class="text-lg">• 線上編輯器</div>
      <div class="absolute top-0 right-0 opacity-10 text-9xl">🖥️</div>
    </div>
    <div v-click class="p-5 border border-yellow-500 rounded-lg relative overflow-hidden">
      <div class="text-2xl font-bold text-yellow-400 mb-4">後端 - Nepeta-cataria</div>
      <div class="text-lg mb-3">• Golang 搭配 Gin 框架</div>
      <div class="text-lg mb-3">• RESTful API 設計</div>
      <div class="text-lg">• 高效能</div>
      <div class="absolute top-0 right-0 opacity-10 text-9xl">⚙️</div>
    </div>

  </div>

  <div>
    <div v-click class="p-5 border border-blue-500 rounded-lg relative overflow-hidden mb-8">
      <div class="text-2xl font-bold text-blue-400 mb-4">容器 - cat-wheel</div>
      <div class="text-lg mb-3">• 隔離的程式執行環境</div>
      <div class="text-lg mb-3">• linux 容器技術</div>
      <div class="text-lg">• 安全的程式碼評測機制</div>
      <div class="absolute top-0 right-0 opacity-10 text-9xl">📦</div>
    </div>
    <div v-click class="p-5 border border-green-500 rounded-lg relative overflow-hidden">
      <div class="text-2xl font-bold text-green-400 mb-4">AI Server - NekoSensei</div>
      <div class="text-lg mb-3">• Python 搭配 FastAPI</div>
      <div class="text-lg mb-3">• 整合大型語言模型</div>
      <div class="text-lg">• 程式碼分析與生成能力</div>
      <div class="absolute top-0 right-0 opacity-10 text-9xl">🤖</div>
    </div>

  </div>
</div>

<style>
h1 {
  font-size: 2.5rem !important;
  font-weight: 600;
  margin-bottom: 1.5rem;
}

.container-animation {
  position: absolute;
  top: 10px;
  right: 10px;
  width: 150px;
  height: 100px;
  opacity: 0.9;
}

.main-container {
  position: absolute;
  top: 50px;
  left: 50px;
  font-size: 40px;
  z-index: 2;
}

</style>

---

```plantuml
@startuml SHARE-CAT Architecture
allow_mixing
!define RECTANGLE class
!define COMPONENT component

skinparam backgroundColor #333333
skinparam ClassBackgroundColor #2B2B2B
skinparam ClassBorderColor #6366F1
skinparam ComponentBackgroundColor #2B2B2B
skinparam ComponentBorderColor #10B981
skinparam ArrowColor #94A3B8
skinparam NoteBorderColor #F59E0B
skinparam NoteBackgroundColor #3F3F3F
skinparam NoteTextColor #FFFFFF
skinparam ClassFontColor #FFFFFF
skinparam ComponentFontColor #FFFFFF

package "Users" {
  RECTANGLE User01
  RECTANGLE User02
  RECTANGLE User03
}

package "Application" {
  package "Tasks" {
    RECTANGLE task01
    RECTANGLE task02
    RECTANGLE task03
  }

  package "Queue Library" #2B593A {
    COMPONENT Producer #593A2B
    COMPONENT "Buffer"  #2B3A59
    COMPONENT "Weighted Round\nRobin Scheduler" #2B593A

    Producer -right-> "Buffer" : push
    "Buffer" -right-> Consumer : pop
  }

  package "Workers" {
    RECTANGLE worker01
    RECTANGLE worker02
    RECTANGLE worker03
    RECTANGLE worker04
  }
}

User01 ..> task01 : Submit
User02 ..> task02 : Submit
User03 ..> task03 : Submit

task01 --> Producer : Enqueue Task
task02 --> Producer : Enqueue Task
task03 --> Producer : Enqueue Task

Consumer --> worker01 : dispatch task
Consumer --> worker02 : dispatch task
Consumer --> worker03 : dispatch task
Consumer --> worker04 : dispatch task

note bottom of Application
  **System Components:**
  **Frontend - Nekopara:** React & Next.js framework
  **Backend - Nepeta-cataria:** Golang with Gin framework
  **Container - cat-wheel:** Docker container technology (elastic scaling)
  **AI Server - NekoSensei:** Python with FastAPI
end note

@enduml
```

---

# 系統部署與未來計劃

<div class="grid grid-cols-2 gap-10 mt-6">
  <div v-click>
    <div class="text-2xl font-bold text-gradient mb-4">開發進度</div>
    <div class="space-y-4">
      <div class="flex items-center">
        <div class="w-3 h-3 rounded-full bg-yellow-400 mr-3"></div>
        <div class="text-xl">前端框架 - 開發中</div>
      </div>
      <div class="flex items-center">
        <div class="w-3 h-3 rounded-full bg-yellow-400 mr-3"></div>
        <div class="text-xl">後端 API - 開發中</div>
      </div>
      <div class="flex items-center">
        <div class="w-3 h-3 rounded-full bg-yellow-400 mr-3"></div>
        <div class="text-xl">AI 系統 - 開發中</div>
      </div>
      <div class="flex items-center">
        <div class="w-3 h-3 rounded-full bg-red-400 mr-3"></div>
        <div class="text-xl">容器系統 - 規劃中</div>
      </div>
    </div>
  </div>

  <div v-click>
    <div class="text-2xl font-bold text-gradient mb-4">未來發展方向</div>
    <div class="space-y-3">
      <div class="flex items-start">
        <div class="text-xl text-blue-400 mr-2">●</div>
        <div class="text-xl">多語言</div>
      </div>
      <div class="flex items-start">
        <div class="text-xl text-blue-400 mr-2">●</div>
        <div class="text-xl">社群功能</div>
      </div>
      <div class="flex items-start">
        <div class="text-xl text-blue-400 mr-2">●</div>
        <div class="text-xl">插件系統</div>
      </div>
    </div>
  </div>
</div>

<div v-click class="mt-8 p-4 bg-gradient-to-r from-purple-900 to-indigo-900 rounded-lg">
  <div class="text-xl font-bold mb-2">預計發布時間表</div>
  <div class="text-lg">預計在這個學期完成MVP版本</div>
</div>

<style>
h1 {
  font-size: 2.5rem !important;
  font-weight: 600;
  margin-bottom: 1.5rem;
}

.text-gradient {
  background: linear-gradient(90deg, #4299e1, #9f7aea);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}
</style>

---

# 謝謝

<div class="mt-6 text-2xl animate-pulse">
有任何問題歡迎提問
</div>

<div class="fixed bottom-10 text-gray-400">
簡報製作: 白雲
</div>

<style>
h1 {
  font-size: 5rem !important;
  font-weight: 600;
  letter-spacing: 2px;
  animation: glow 1.5s ease-in-out infinite alternate;
}

@keyframes glow {
  from {
    text-shadow: 0 0 5px #fff, 0 0 10px #fff, 0 0 15px #0073e6, 0 0 20px #0073e6;
  }
  to {
    text-shadow: 0 0 10px #fff, 0 0 20px #fff, 0 0 30px #0073e6, 0 0 40px #0073e6;
  }
}

.animate-pulse {
  animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}

@keyframes pulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.5;
  }
}
</style>
