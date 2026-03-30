# 論文筆記：LongMemEval

**標題：** LONGMEMEVAL: BENCHMARKING CHAT ASSISTANTS ON LONG-TERM INTERACTIVE MEMORY
**作者：** Di Wu, Hongwei Wang, Wenhao Yu, Yuwei Zhang, Kai-Wei Chang, Dong Yu
**發表：** ICLR 2025 | arXiv:2410.10813
**論文連結：** https://arxiv.org/pdf/2410.10813
**程式碼：** https://github.com/xiaowu0162/LongMemEval

---

## 問題與動機

現代 LLM chat assistant 被期望能記住跨越多個 session 的使用者資訊（偏好、生活事件、知識更新等），但目前缺乏嚴謹的長期記憶評估 benchmark。

實測發現商業系統存在巨大落差：
- **ChatGPT**（online memory 模式）：準確率 57.7%，vs. 離線全文閱讀 91.8%，落差 **37%**
- **Coze**：準確率 33%，vs. 離線 92%，落差 **64%**

現有 benchmark 的局限：
- Context 長度上限不足（MemoryBank、PerLTQA、LoCoMo、DialSim 最多 1–350k tokens）
- 使用閒聊對話（chit-chat），不夠貼近真實任務場景
- 未涵蓋知識更新（Knowledge Update）與拒答（Abstention）等關鍵能力

---

## 核心貢獻

1. **LongMemEval benchmark**：500 道人工審核題目，嵌入可自由擴展的對話歷史，涵蓋五種記憶能力 × 七種題型
2. **兩個 benchmark 變體**：
   - **LongMemEvalS**：~115k tokens/history（短版）
   - **LongMemEvalM**：500 sessions，~1.5M tokens（大規模多 session）
3. **統一記憶增強 assistant 框架**：Indexing → Retrieval → Reading 三階段，四個關鍵控制點（CP）
4. **系統性優化策略**：session 分解、事實增強索引擴展、時間感知查詢擴展、結構化 Chain-of-Note 閱讀

---

## 資料集情境

**模擬場景**：使用者與 AI chat assistant 進行**跨多個 session 的長期任務對話**，累積時間跨越數週至數月。

**參與者**：
- 單一持久使用者（具備豐富合成個人背景）
- AI chat assistant（被測試是否能記住使用者歷史資訊）

**對話性質**：
- **任務導向**（非閒聊）：使用者在完成實際任務時（如規劃旅遊、尋求推薦、解決問題）自然透露個人資訊
- 個人資訊**間接嵌入**：不是直接陳述，而是作為任務脈絡的副產品出現（例如討論旅遊計畫時提到「我對海鮮過敏」），大幅增加記憶難度
- 證據 sessions 隨機散落在大量無關 sessions 之中，類似「大海撈針」設定

**使用者屬性本體論（164 個屬性，五大類）**：
- 人口統計資訊（年齡、職業、居住地）
- 生活方式（飲食習慣、運動偏好）
- 生活事件（換工作、搬家、結婚）
- 當前情境（目前計畫、近期狀況）
- 個人物品（車、寵物、設備）

> **一句話情境**：使用者在與 AI 助理完成各種日常任務的過程中，隨對話自然揭露個人資訊，assistant 需在未來 session 中準確記起並運用這些資訊。

---

## 資料集建構

### 三階段建構流程

**Stage 1 — 題目策劃（Question Curation）**
- LLM 以使用者屬性為基礎生成題目種子
- 人工專家過濾並改寫，確保品質、自然度與無歧義
- 同時收集標準答案

**Stage 2 — 證據 Session 建構**
- 將相關個人資訊透過「self-chatting」（LLM 同時扮演使用者與 assistant）嵌入任務對話
- 資訊間接嵌入，非直接陳述
- 每道題目的證據可分散在**最多 6 個不同 session**

**Stage 3 — 歷史對話彙整**
- 將證據 sessions 隨機插入從 ShareGPT、UltraChat 取樣的無關 sessions 中
- 多樣化證據位置與密度

### 資料集規模

| 指標 | 數值 |
|------|------|
| 總題目數 | 500 |
| 題型數 | 7 |
| 拒答題（Abstention） | 30 |
| LongMemEvalS 長度 | ~115k tokens |
| LongMemEvalM 長度 | ~1.5M tokens（500 sessions） |

---

## 五種記憶能力 × 七種題型

| 記憶能力 | 說明 |
|---------|------|
| **Information Extraction (IE)** | 從歷史中精確回憶特定事實 |
| **Multi-Session Reasoning (MR)** | 整合來自多個 sessions 的資訊進行推理 |
| **Knowledge Updates (KU)** | 識別使用者資訊已更新，反映最新狀態（而非舊版本） |
| **Temporal Reasoning (TR)** | 理解時間維度（例如「上個月使用者提到了什麼？」） |
| **Abstention (ABS)** | 識別問題無法從歷史中回答（含有錯誤前提），正確拒答而非幻覺 |

---

## 評估指標

- **Answer Accuracy**：以 GPT-4o 作為評判者 (LLM-as-judge)，對比 assistant 回答與標準答案，人機一致率 >97%
- **Memory Recall**：Recall@k 與 NDCG@k，對照人工標注的證據 session 位置，獨立評估取回品質

---

## 主要實驗結果

### 商業系統基線

| 系統 | 長期記憶模式準確率 | 離線全文閱讀準確率 | 落差 |
|------|-----------------|-----------------|------|
| ChatGPT | 57.7% | 91.8% | **-37%** |
| Coze | 33.0% | 92.0% | **-64%** |

根本原因：間接陳述的資訊未被捕捉；Knowledge Update 處理不當（記住舊版本）。

### 長文本 LLM（餵入完整歷史）

| 模型 | Oracle 準確率 | 全文輸入準確率 | 落差 |
|------|-------------|-------------|------|
| GPT-4o | ~90% | ~60% | **-30%** |
| Llama 3.1 70B | ~88% | ~33% | **-55%** |
| Phi-3.5 Mini | ~82% | ~34% | **-48%** |

### 四個控制點（CP）的優化效果

| 控制點 | 優化策略 | 效果 |
|--------|---------|------|
| **CP1 Value（儲存粒度）** | Session → 單輪對話分解 | 最有效的單項改進 |
| **CP2 Key（索引策略）** | 事實增強索引擴展（加入提取的事實作為額外索引鍵） | +9.4% Recall@k，+5.4% 最終準確率 |
| **CP3 Query（時間感知）** | 用 GPT-4o 解析時間線索，生成時間範圍限制查詢 | 時間推理 Recall +6.8%–11.3% |
| **CP4 Reading（閱讀策略）** | 結構化 Chain-of-Note（JSON 格式：先提取再推理） | +最多 10 個百分點 |

---

## 結論與啟示

1. **現有最強商業與開源系統都顯著不足**：在長期多 session 記憶上，準確率落差達 30–64%
2. **Session 分解是最關鍵的單項設計選擇**：細粒度儲存遠優於整段 session
3. **事實增強索引大幅改善取回**：語義搜尋 + 結構化事實雙路徑互補
4. **時間感知查詢不可缺**：但需要夠強的 LLM，弱模型容易幻覺時間範圍
5. **結構化閱讀（Chain-of-Note）普遍有效**：各模型規模均提升

### 局限
- 資料集為合成對話，個人資訊透過 LLM 生成（雖有人工審核）
- 500 題規模相對有限
- 英文為主

---

## 與其他 Benchmark 的關係

| 面向 | LongMemEval | LoCoMo | MemBench | MemoryAgentBench |
|------|------------|--------|----------|-----------------|
| 情境 | 任務導向對話（間接資訊嵌入） | 朋友日常閒聊 | 個人助理對話 / 被動旁觀 | 多種異質情境 |
| Context 規模 | 115k / 1.5M tokens | ~9K tokens | 最大 ~100k tokens | 最大 1.44M tokens |
| 拒答能力 | ✓（30 題） | ✗ | ✗ | ✗ |
| 知識更新 | ✓ | ✗ | ✓（Knowledge-updating 子類型） | ✓（FactConsolidation） |
| 題目人工審核 | ✓（全部 500 題） | 部分 | ✗（全自動） | 部分 |
