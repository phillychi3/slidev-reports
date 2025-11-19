---
background: https://api.cloudowo.com/api/v1/pixiv/image/89720361
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

## 智能桌面寵物 - 11/19專題報告

<div class="pt-12">
  <span class="px-2 py-1 rounded cursor-pointer"
        hover="bg-white bg-opacity-10">
    凱爾希也算是個人助理吧(爆論
  </span>
</div>


---
class: py-10
---

# WAIFU LAB
<div class="flex flex-col items-center">
    <img src="/167647828.png"/>
</div>


---
class: flex justify-center items-center
---

<div
  absolute text-6xl
  :class="$clicks < 1 ? 'text-white' : 'translate-y--18 scale-40 text-white/30'"
  transition duration-500 ease-in-out
>
  <span>當我想要開始的時候</span>
</div>

<div flex flex-col items-center>
  <v-clicks>
    <div mt-4>
      <h1 flex items-center text="5xl!">
        <span class='text-white'>這文檔是哪個天才寫的?</span>
      </h1>
    </div>
  </v-clicks>
</div>

---
class: 'text-center'
---

# langchain

<div class="mt-6 text-left gap-4 flex flex-col">
<v-clicks>
<div border="2 solid blue-800/80" rounded-lg>
  <div bg="blue-800/30" px-4 py-3>
    <div>
      <span>
        1. 複雜性與可維護性, 數據經多個抽象層, 難以追蹤邏輯
      </span>
    </div>
  </div>
</div>
<div border="2 solid blue-800/80" rounded-lg>
  <div bg="blue-800/30" px-4 py-3>
    <div>
      <span>
        2. 限制客製化能力, 難以實現高度客製化的LLM工作
      </span>
    </div>
  </div>
</div>

</v-clicks>
</div>

---
class: flex justify-center items-center
---

<div
  absolute text-6xl
  :class="$clicks < 1 ? 'text-white' : 'translate-y--18 scale-40 text-white/30'"
  transition duration-500 ease-in-out
>
  <span>這時就該做某件事情了</span>
</div>

<div flex flex-col items-center>
  <v-clicks>
    <div mt-4>
      <h1 flex items-center text="5xl!">
        <span class='text-white'>造輪子!</span>
      </h1>
    </div>
  </v-clicks>
</div>

---
layout: intro
class: px-24
glowSeed: 205
---

<div class="flex items-center justify-center h-full">
  <div class="relative flex items-center text-9xl font-bold">
    <div
      v-motion
      :initial="{ x: 0 }"
      :enter="{ x: 230, transition: { delay: 500, duration: 800 } }"
      class=" bg-clip-text text-transparent text-white"
    >
      H
    </div>
    <div
      v-motion
      :initial="{ opacity: 1, x: 0, scale: 1 }"
      :enter="{ opacity: 0, x: -100, scale: 0, transition: { delay: 500, duration: 800 } }"
      class= "bg-clip-text text-transparent text-white"
    >
      ydrogen
    </div>
    <div
      v-motion
      :initial="{ opacity: 1, x: 0 }"
      :enter="{ x: -255, transition: { delay: 500, duration: 800 } }"
      class="ml-4 bg-clip-text text-transparent text-white"
    >
      AI
    </div>
  </div>
</div>

---
class: py-10
---

# Flow

````md magic-move
```python
review = ReviewNode("review")
payment = PaymentNode("payment")
finish = FinishNode("finish")

review >> payment >> finish

```

```python
review2 = ReviewNode("review")
payment2 = PaymentNode("payment")
revise2 = ReviseNode("revise")
finish2 = FinishNode("finish")

review2 >> {
    "approved": payment2,
    "needs_revision": revise2,
    "rejected": finish2
}

payment2 >> finish2
revise2 >> review2

```
````


---
class: py-10
---

# 什麼是Hydrogen AI


<div mt-2 />

<div :class="$clicks == 0 ? 'text-white' : 'hidden'" class="mb-6">
  <div text-lg font-bold mb-3 flex items-center>
    LangChain
  </div>
</div>
<div :class="$clicks == 1 ? 'text-white' : 'hidden'" class="mb-6">
  <div text-lg font-bold mb-3 flex items-center>
    Haystack
  </div>
</div>
<div :class="$clicks == 2 ? 'text-white' : 'hidden'" class="mb-6">
  <div text-lg font-bold mb-3 flex items-center>
    HAI
  </div>
</div>

````md magic-move
```python

qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=vectordb.as_retriever(search_kwargs={"k": 3}),
    chain_type_kwargs={"prompt": prompt},
    return_source_documents=True
)

result = qa_chain({"query": "你的問題"})
print(f"答案: {result['result']}")
print(f"來源文件: {result['source_documents']}")
```

```python
rag_pipeline = Pipeline()
rag_pipeline.add_component("text_embedder", OpenAITextEmbedder())
rag_pipeline.add_component("retriever", InMemoryEmbeddingRetriever(document_store=document_store, top_k=3))
rag_pipeline.add_component("prompt_builder", prompt_builder)
rag_pipeline.add_component("llm", OpenAIGenerator(model="gpt-4"))

rag_pipeline.connect("text_embedder.embedding", "retriever.query_embedding")
rag_pipeline.connect("retriever.documents", "prompt_builder.documents")
rag_pipeline.connect("prompt_builder", "llm")

result = rag_pipeline.run({
    "text_embedder": {"text": "你的問題"},
    "prompt_builder": {"question": "你的問題"}
})

print(result["llm"]["replies"][0])
```

```python
query_embed >> retriever >> context_builder

context = {"query": "文件中提到了哪些內容？"}

current = query_embed
while current:
    current = await current.execute(context)

print(f"檢索到 {context['retrieved_count']} 個相關文件")
```
````



---
class: py-10
---
# 目前已實現功能

<div grid grid-cols-5 gap-4>
  <div v-click="1" class="rounded-lg p-12 bg-blue-900/20 text-center flex flex-col items-center gap-2">
    <div text-xl font-bold text-nowrap text-blue-200>LLM Providers</div>
    <div text-xs text-nowrap text-blue-200 mt-1>openai, gemini, Anthropic...</div>
  </div>

  <div v-click="2" class="rounded-lg p-12 bg-blue-800/20 text-center flex flex-col items-center gap-2">
    <div text-xl font-bold text-nowrap text-blue-100 mt-1>Loaders</div>
    <div text-xs text-nowrap text-blue-100 mt-1>text , markdown, web</div>
  </div>

  <div v-click="3" class="rounded-lg p-12 bg-indigo-900/20 text-center flex flex-col items-center gap-2">
    <div text-xl font-bold text-nowrap text-indigo-200 mt-1>Splitters</div>
    <div text-xs text-nowrap text-indigo-200 mt-1>RecursiveSplitter, Splitter</div>
  </div>

  <div v-click="4" class="rounded-lg p-12 bg-indigo-800/20 text-center flex flex-col items-center gap-2">
    <div text-xl font-bold text-nowrap text-indigo-100 mt-1>Embedders</div>
    <div text-xs text-nowrap text-indigo-100 mt-1>openai</div>
  </div>

  <div v-click="5" class="rounded-lg p-12 bg-cyan-900/20 text-center flex flex-col items-center gap-2">
    <div text-xl font-bold text-nowrap text-cyan-200 mt-1>Vector Stores</div>
    <div text-xs text-nowrap text-cyan-200 mt-1>qdrant</div>
  </div>

  <div v-click="6" class="rounded-lg p-12 bg-cyan-800/20 text-center flex flex-col items-center gap-2">
    <div text-xl font-bold text-nowrap text-cyan-100 mt-1>Retrievers</div>
    <div text-xs text-nowrap text-cyan-100 mt-1>qdrant</div>
  </div>
  <div v-click="7" class="rounded-lg p-12 bg-sky-900/20 text-center flex flex-col items-center gap-2">
    <div text-xl font-bold text-nowrap text-sky-200 mt-1>preprocessor</div>
    <div text-xs text-nowrap text-sky-200 mt-1>cleaner</div>
  </div>
  <div v-click="8" class="rounded-lg p-12 bg-sky-800/20 text-center flex flex-col items-center gap-2">
    <div text-xl font-bold text-nowrap text-sky-100 mt-1>graphstores</div>
    <div text-xs text-nowrap text-sky-100 mt-1>neo4j</div>
  </div>
</div>


---
class: py-10
---


# 未來規劃
<div class="grid grid-cols-2 gap-6">
<div class="bg-green/10 p-6 rounded-lg flex flex-col gap-4">
    <span class="text-yellow">主要路線</span>
    <div class="flex flex-col gap-2">
        <span>新增更多模型與資料庫支援</span>
        <span>新增logging</span>
        <span>編寫test</span>
        <span>添加observability支援</span>
    </div>
</div>

<div class="bg-green/10 p-6 rounded-lg flex flex-col gap-4">
    <span class="text-yellow">支線</span>
    <div class="flex flex-col gap-2">
        <span>文檔網站</span>
        <span>開源前處理</span>
    </div>
</div>
</div>



---
layout: image
image: ./ygg.png
---

<div class="flex items-center justify-center h-full">
    <span class=" text-7xl text-black">歷史樹系統</span>
</div>

---
class: py-10
---



<div mt-8 />

<div text-center mb-12>
  <h1 text-4xl font-bold mb-4>HistoryTree 系統架構</h1>
  <p text-xl text-gray-400>時態知識管理 RAG 系統</p>
</div>

<div grid grid-cols-3 gap-8>
  <div v-click="1">
    <div text-xl font-bold mb-6 flex items-center>
      <div i-carbon:document text-2xl mr-3 text-blue-400 />
      文檔處理層
    </div>
    <div space-y-5 text-base>
      <div flex items-center>
        <div i-carbon:cloud-upload mr-3 text-blue-300 />
        <span>Document Loader</span>
      </div>
      <div flex items-center>
        <div i-carbon:split mr-3 text-purple-300 />
        <span>Chunker (20% overlap)</span>
      </div>
      <div flex items-center>
        <div i-carbon:ai-results mr-3 text-green-300 />
        <span>Fact Extractor (LLM)</span>
      </div>
      <div flex items-center>
        <div i-carbon:connect mr-3 text-yellow-300 />
        <span>Entity Resolver</span>
      </div>
    </div>
  </div>

  <div v-click="2">
    <div text-xl font-bold mb-6 flex items-center>
      <div i-carbon:data-base text-2xl mr-3 text-green-400 />
      儲存層
    </div>
    <div space-y-5 text-base>
      <div>
        <div flex items-center mb-2>
          <div i-carbon:cube mr-3 text-cyan-300 />
          <span font-bold>Vector Store</span>
        </div>
        <div ml-8 text-sm text-gray-400>
          <div>• Qdrant</div>
          <div>• 語義搜尋</div>
          <div>• 知識檢索</div>
        </div>
      </div>
      <div>
        <div flex items-center mb-2>
          <div i-carbon:network-3 mr-3 text-pink-300 />
          <span font-bold>Graph Store</span>
        </div>
        <div ml-8 text-sm text-gray-400>
          <div>• Neo4j</div>
          <div>• 事實關係</div>
          <div>• 時態版本</div>
        </div>
      </div>
    </div>
  </div>

  <div v-click="3">
    <div text-xl font-bold mb-6 flex items-center>
      <div i-carbon:search text-2xl mr-3 text-purple-400 />
      檢索層
    </div>
    <div space-y-5 text-base>
      <div>
        <div flex items-center mb-2>
          <div i-carbon:layers mr-3 text-orange-300 />
          <span font-bold>Layer 1: 低層檢索</span>
        </div>
        <div ml-8 text-sm text-gray-400>
          <div>• 向量相似度</div>
          <div>• BM25 關鍵字</div>
        </div>
      </div>
      <div>
        <div flex items-center mb-2>
          <div i-carbon:chart-network mr-3 text-blue-300 />
          <span font-bold>Layer 2: 高層檢索</span>
        </div>
        <div ml-8 text-sm text-gray-400>
          <div>• 圖遍歷</div>
          <div>• 知識推理</div>
        </div>
      </div>
    </div>
  </div>
</div>


---
class: py-10
---


<div mt-12 v-click="1">
  <div text-xl font-bold mb-6 text-center flex items-center justify-center>
    <div i-carbon:application text-2xl mr-3  />
    核心技術棧
  </div>
  <div grid grid-cols-4 gap-6 text-base>
    <div text-center p-4 border border-gray-700 rounded-lg>
      <div>GPT-4o-mini</div>
      <div text-sm text-gray-400>事實抽取</div>
    </div>
    <div text-center p-4 border border-gray-700 rounded-lg>
      <div>Qdrant</div>
      <div text-sm text-gray-400>向量存儲</div>
    </div>
    <div text-center p-4 border border-gray-700 rounded-lg>
      <div>Neo4j</div>
      <div text-sm text-gray-400>圖數據庫</div>
    </div>
    <div text-center p-4 border border-gray-700 rounded-lg>
      <div>FastAPI</div>
      <div text-sm text-gray-400>REST API</div>
    </div>
  </div>
</div>
<div mt-12 v-click="2">
    <div text-center p-6 bg-gradient-to-r rounded-lg>
        <div text-lg font-bold mb-3>資料流程</div>
        <div flex items-center justify-center gap-4>
            <div border border-gray-700 rounded-lg p-4 text-nowrap>文檔</div>
            <div>→</div>
            <div border border-gray-700 rounded-lg p-4 text-nowrap>分塊</div>
            <div>→</div>
            <div border border-gray-700 rounded-lg p-4 text-nowrap>LLM 抽取事實</div>
            <div>→</div>
            <div border border-gray-700 rounded-lg p-4 text-nowrap>詞義消歧</div>
                <div>→</div>
        </div>
        <div flex items-center justify-center gap-4 mt-4>
            <div border border-gray-700 rounded-lg p-4 text-nowrap>嵌入</div>
            <div>→</div>
            <div border border-gray-700 rounded-lg p-4 text-nowrap>雙存儲</div>
            <div>→</div>
            <div border border-gray-700 rounded-lg p-4 text-nowrap>雙層檢索</div>
            <div>→</div>
            <div border border-gray-700 rounded-lg p-4 text-nowrap>結果融合</div>
        </div>
    </div>
</div>


---
class: py-10
---
# 詳細實現請見賴瑞晨簡報


---
class: py-10
---
# 1..程式碼

```python
loader = TextLoader(name="loader", source=file_path)
chunker = ChunkProcessor(name="chunker", splitter=self.splitter)
fact_extractor = FactExtractor(name="fact_extractor", llm=self.llm)
entity_resolver = EntityResolver(name="entity_resolver")
embedding_processor = EmbeddingProcessor(
    name="embedder",
    embedder=self.embedder,
    batch_size=32,
)
vector_writer = QdrantStoreWriter(
    name="vector_writer",
    store=self.vector_store,
)
entity_linker = EntityLinker(
    name="entity_linker",
    graph_store=self.graph_store,
)
(
    loader
    >> chunker
    >> fact_extractor
    >> entity_resolver
    >> embedding_processor
    >> vector_writer
    >> entity_linker
) # type: ignore

current = loader
while current:
    current = await current.execute(ingest_context)
```

---
class: py-10
---
# 測試資料 感謝G老師生成

```markdown
張三是一位資深軟體工程師，目前在台北科技公司工作。他在2020年加入這家公司，擔任後端開發的職位。

張三畢業於台灣大學資訊工程系，擁有10年的軟體開發經驗。他專精於Python和Go語言，並且對分散式系統有深入的研究。

台北科技公司是一家專注於AI技術的新創公司，成立於2018年，總部位於台北市信義區。公司目前有50名員工，主要業務是開發企業級AI解決方案。

張三的直屬主管是李四，李四是公司的技術長(CTO)。李四在創立公司之前曾在Google工作5年，負責機器學習相關專案。

張三最近正在開發一個名為HistoryTree的RAG系統，這個系統結合了向量資料庫和圖資料庫，用於時態知識管理。該專案使用Neo4j作為圖資料庫，Qdrant作為向量資料庫。

公司的辦公室位於台北101附近，交通便利。張三每天搭乘捷運上班，通勤時間約30分鐘。

張三的技術部落格很受歡迎，他經常分享關於RAG、知識圖譜和大語言模型的技術文章。他的部落格每月有超過10000次瀏覽。

在業餘時間，張三喜歡參加技術社群活動，他是PyData Taipei的活躍成員，並且在2023年擔任過演講者，分享關於向量資料庫優化的經驗。
```

---
class: py-10
---


## 實際效果


<img src="./2025-11-19-062250.png" class=" h-full mx-auto ">


---
class: py-10
---


## 實現?
```bash
Initializing HistoryTree...

Ingesting document...
✓ Loaded 1 documents
✓ Created 1 chunks
✓ Extracted 27 facts
✓ Extracted 11 entities
✓ Stored 1 chunks in vector DB
✓ Added 11 entities to graph
✓ Added 27 facts to graph

Searching...
✓ Embedded query
✓ Retrieved 1 documents from vector store

Search Results:

  Result 1:
    Score: 0.526
    Content: 張三是一位資深軟體工程師，目前在台北科技公司工作。他在2020年加入這家公司，擔任後端開發的職位。

張三畢業於台灣大學資訊工程系，擁有10年的軟體開發經驗。他專精於Python和Go語言，並且對分散...
✓ Closed connections
```

---
class: py-10
---


## 還有哪些部分正在進行

<img src="./ailaunacher.png" class=" h-full mx-auto ">


---
layout: center
---

# AI VTuber Launcher 後端架構

<div mt-8 />

<div grid grid-cols-2 gap-12>
  <div v-click="1">
    <div text-xl font-bold mb-6 flex items-center>
      <div i-carbon:application text-2xl mr-3 />
      核心框架
    </div>
    <div space-y-6 text-lg>
      <div flex items-center text="[28px]">
        <div i-simple-icons:rust mr-3 text-orange-500 />
        <span>Tauri 2.x</span>
      </div>
      <div flex items-center text="[28px]">
        <div i-carbon:data-base mr-3 text-blue-400 />
        <span>bincode</span>
      </div>
      <div flex items-center text="[28px]">
        <div i-carbon:plug mr-3 text-purple-400 />
        <span>Plugin System</span>
      </div>
    </div>
  </div>

  <div v-click="2">
    <div text-xl font-bold mb-6 flex items-center>
      <div i-carbon:flow-data text-2xl mr-3 />
      狀態管理
    </div>
    <div space-y-6 text-lg>
      <div flex items-center text="[28px]">
        <div i-carbon:locked mr-3 text-green-400 />
        <span>parking_lot::RwLock</span>
      </div>
      <div flex items-center text="[28px]">
        <div i-carbon:cube mr-3 text-cyan-400 />
        <span>Arc (共享狀態)</span>
      </div>
      <div flex items-center text="[28px]">
        <div i-carbon:event mr-3 text-yellow-400 />
        <span>EventBus</span>
      </div>
    </div>
  </div>
</div>


---
layout: center
---

# 插件系統架構

<div mt-8 />

<div grid grid-cols-2 gap-12>
  <div v-click="1">
    <div text-xl font-bold mb-6 flex items-center>
      <div i-carbon:plug text-2xl mr-3 />
      動態載入
    </div>
    <div space-y-6 text-lg>
      <div flex items-center text="[28px]">
        <div i-carbon:assembly-cluster mr-3 text-purple-400 />
        <span>libloading</span>
      </div>
      <div flex items-center text="[28px]">
        <div i-carbon:data-structured mr-3 text-blue-400 />
        <span>Plugin Trait</span>
      </div>
      <div flex items-center text="[28px]">
        <div i-carbon:box mr-3 text-green-400 />
        <span>FFI (cdylib)</span>
      </div>
    </div>
  </div>

  <div v-click="2">
    <div text-xl font-bold mb-6 flex items-center>
      <div i-carbon:api text-2xl mr-3 />
      插件接口
    </div>
    <div space-y-5 text-base>
      <div flex items-center>
        <div i-carbon:play-outline mr-3 text-green-300 />
        <span>initialize / start / stop</span>
      </div>
      <div flex items-center>
        <div i-carbon:settings-adjust mr-3 text-blue-300 />
        <span>get/update_settings</span>
      </div>
      <div flex items-center>
        <div i-carbon:event mr-3 text-yellow-300 />
        <span>Event Handlers</span>
      </div>
      <div flex items-center>
        <div i-carbon:view mr-3 text-purple-300 />
        <span>UI Schema</span>
      </div>
    </div>
  </div>
</div>

---
layout: center
---

# 啟動器架構

<div mt-8 />

<div grid grid-cols-3 gap-8>
  <div v-click="1">
    <div text-lg font-bold mb-5 flex items-center justify-center>
      <div i-carbon:video mr-2 text-purple-400 />
      <span>直播模式</span>
    </div>
    <div space-y-4 text-sm>
      <div flex items-center>
        <div i-carbon:video-chat mr-2 text-purple-300 />
        <span>OBS 整合</span>
      </div>
      <div flex items-center>
        <div i-carbon:character-patterns mr-2 text-pink-300 />
        <span>Live2D 渲染</span>
      </div>
      <div flex items-center>
        <div i-carbon:microphone mr-2 text-blue-300 />
        <span>語音輸入</span>
      </div>
    </div>
  </div>

  <div v-click="2">
    <div text-lg font-bold mb-5 flex items-center justify-center>
      <div i-carbon:laptop mr-2 text-blue-400 />
      <span>桌面寵物</span>
    </div>
    <div space-y-4 text-sm>
      <div flex items-center>
        <div i-carbon:overlay mr-2 text-blue-300 />
        <span>桌面覆蓋</span>
      </div>
      <div flex items-center>
        <div i-carbon:cursor-2 mr-2 text-green-300 />
        <span>互動系統</span>
      </div>
      <div flex items-center>
        <div i-carbon:ai-status mr-2 text-yellow-300 />
        <span>AI 對話</span>
      </div>
    </div>
  </div>

  <div v-click="3">
    <div text-lg font-bold mb-5 flex items-center justify-center>
      <div i-carbon:chat mr-2 text-indigo-400 />
      <span>Discord Bot</span>
    </div>
    <div space-y-4 text-sm>
      <div flex items-center>
        <div i-carbon:bot mr-2 text-indigo-300 />
        <span>Discord API</span>
      </div>
      <div flex items-center>
        <div i-carbon:send-alt mr-2 text-blue-300 />
        <span>訊息處理</span>
      </div>
      <div flex items-center>
        <div i-carbon:voice-activate mr-2 text-purple-300 />
        <span>語音頻道</span>
      </div>
    </div>
  </div>
</div>


---
layout: center
---

# 技術特點

<div mt-8 />

<div grid grid-cols-2 gap-12>
  <div v-click="1">
    <div text-xl font-bold mb-6 flex items-center>
      <div i-carbon:recently-viewed text-2xl mr-3 />
      並發安全
    </div>
    <div space-y-5 text-base>
      <div flex items-center>
        <div i-carbon:locked mr-3 text-green-400 />
        <span>parking_lot::RwLock 無死鎖</span>
      </div>
      <div flex items-center>
        <div i-carbon:data-share mr-3 text-blue-400 />
        <span>Arc 共享所有權</span>
      </div>
      <div flex items-center>
        <div i-carbon:flow mr-3 text-purple-400 />
        <span>crossbeam 通道通訊</span>
      </div>
    </div>
  </div>

  <div v-click="2">
    <div text-xl font-bold mb-6 flex items-center>
      <div i-carbon:plug text-2xl mr-3 />
      可擴展性
    </div>
    <div space-y-5 text-base>
      <div flex items-center>
        <div i-carbon:catalog mr-3 text-purple-400 />
        <span>動態插件載入</span>
      </div>
      <div flex items-center>
        <div i-carbon:event mr-3 text-yellow-400 />
        <span>事件驅動架構</span>
      </div>
      <div flex items-center>
        <div i-carbon:api-1 mr-3 text-cyan-400 />
        <span>統一 Plugin Trait</span>
      </div>
    </div>
  </div>
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

