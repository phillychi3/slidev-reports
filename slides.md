---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides, markdown enabled
title: Data stonks
info: |
  ## Data stonks
  Lazy lol
  phillychi3 | 戚雲飛
# apply any unocss classes to the current slide
class: text-center
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# https://sli.dev/guide/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
preload: false
---

# DATA STONKS

## 資料整理系統

B11123213 戚雲飛

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    進入下一頁 <carbon:arrow-right class="inline"/>
  </span>
</div>

---

# 介紹

關於Data stonks

<div class="bg-white border border-gray-300 shadow-lg rounded-lg p-4">
  <div class="drop-area bg-gray-100 border border-gray-300 rounded-lg p-4 text-center h-200px">
    <div class="flex justify-center items-center h-full text-dark">
      Drop file here
    </div>
  </div>
</div>

<img
  v-click
  v-motion
  :initial="{ x: 500, y: 800, scale: 2, rotate: 100 }"
  :click-1="final"
  class="absolute inset-0"
  src="https://cdn-icons-png.flaticon.com/512/4248/4248321.png"
  alt=""
  width="100"
  height="100"
/>
<img
  v-motion
  :initial="{ x: 500, y: -200, scale: 2, rotate: 100 }"
  :click-1="final2"
  class="absolute inset-0"
  src="https://cdn-icons-png.flaticon.com/512/186/186320.png"
  alt=""
  width="100"
  height="100"
/>
<img
  v-motion
  :initial="{ x: -100, y: -200, scale: 2, rotate: 100 }"
  :click-1="final3"
  class="absolute inset-0"
  src="https://cdn-icons-png.flaticon.com/512/8760/8760611.png"
  alt=""
  width="100"
  height="100"
/>

<script setup lang="ts">
const final = {
  x: 400,
  y: 200,
  rotate: 0,
  scale: 1,
  transition: {
    type: 'spring',
    damping: 10,
    stiffness: 20,
    mass: 2
  }
}
const final2 = {
  x: 500,
  y: 200,
  rotate: 0,
  scale: 1,
  transition: {
    type: 'spring',
    damping: 10,
    stiffness: 20,
    mass: 2
  }
}
const final3 = {
  x: 300,
  y: 200,
  rotate: 0,
  scale: 1,
  transition: {
    type: 'spring',
    damping: 10,
    stiffness: 20,
    mass: 2
  }
}
</script>

---
---

# 細節

<div class="flex justify-center items-center w-full h h-20vh">
  <div class="flex justify-between items-center">
    <div class="box bg-cyan-500 h-32 w-60 rounded flex justify-center items-center">輸入</div>
    <div class="line w-1/4 h-1 bg-gradient-to-r from-transparent via-white to-transparent animate-slide"></div>
    <div class="box bg-cyan-500 h-32 w-60 rounded flex justify-center items-center">提取資料</div>
    <div class="line w-1/4 h-1 bg-gradient-to-r from-transparent via-white to-transparent animate-slide"></div>
    <div class="box bg-cyan-500 h-32 w-60 rounded flex justify-center items-center">產生標籤</div>
    <div class="line w-1/4 h-1 bg-gradient-to-r from-transparent via-white to-transparent animate-slide"></div>
    <div class="box bg-cyan-500 h-32 w-60 rounded flex justify-center items-center">歸檔</div>
  </div>
</div>

<div
  v-click=[0,1]
  v-motion
  class="absolute box h-25 w-25 bg-cyan-500 flex justify-center items-center rounded-lg"
  :initial="{ x: -80, y: 80 }"
  :enter="{x:-20}"
  :click-1={x:0}
>
  輸入: 網址
</div>
<div
  v-click=[1,2]
  v-motion
  class="absolute box h-25 w-25 bg-cyan-500 flex justify-center items-center rounded-lg"
  :initial="{ x: 0, y: 80 }"
  :enter="{x:0}"
  :click-1={x:260}
>
  get
</div>
<div
  v-click=[2,3]
  v-motion
  class="absolute box h-25 w-25 bg-cyan-500 flex justify-center items-center break-all rounded-lg"
  :initial="{ x: 260, y: 80 }"
  :enter="{x:260}"
  :click-1={x:530}
>
  anime
  ヨルクラ
</div>
<div
  v-click=[3,4]
  v-motion
  class="absolute box h-25 w-25 bg-cyan-500 flex justify-center items-center rounded-lg"
  :initial="{ x: 530, y: 80 }"
  :enter="{x:530}"
  :click-1={x:810}
>

  塞入資料庫
</div>



<style>
  @keyframes slide {
    0% {
      background-position: 100%;
    }
    100% {
      background-position: -100%;
    }
  }

  .animate-slide {
    background: linear-gradient(90deg, transparent, #1abc9c, transparent);
    background-size: 200% 100%;
    animation: slide 1s ease-in-out infinite;
  }
</style>


---
---

# 展示

<table class="table-auto w-full text-left whitespace-nowrap">
  <thead class="border-b border-gray-200 bg-gray-500">
    <tr>
      <th class="px-4 py-2">名稱</th>
      <th class="px-4 py-2">網址</th>
      <th class="px-4 py-2">標籤</th>
    </tr>
  </thead>
  <tbody>
    <tr class="hover:bg-gray-400">
      <td class="px-4 py-2">【試片】《夜晚的水母不會游泳》</td>
      <td class="px-4 py-2"><a href="https://gnn.gamer.com.tw/detail.php?sn=266390" class="text-blue-500 hover:underline">https://gnn.gamer.com.tw/detail.php?sn=266390</a></td>
      <td class="px-4 py-2">anime,ヨルクラ,新聞</td>
    </tr>
  </tbody>
</table>

---
layout: image-right
image: "https://i.pinimg.com/564x/68/65/aa/6865aa43ea77c49773f948d71e428cf8.jpg"
---

# 問題

##### 網頁會遇到爬蟲問題

各種檔案格式讀取

圖片內容提取與分析



---
layout: center
class: text-center
---

# 感謝聆聽