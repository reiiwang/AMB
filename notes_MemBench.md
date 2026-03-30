# 論文筆記：MemBench

**標題：** Towards More Comprehensive Evaluation on the Memory of LLM-based Agents
**作者：** Haoran Tan, Zeyu Zhang, Chen Ma, Xu Chen（人民大學）；Quanyu Dai, Zhenhua Dong（華為諾亞方舟實驗室）
**發表：** Findings of ACL 2025，頁 19336–19352
**論文連結：** https://aclanthology.org/2025.findings-acl.989.pdf
**程式碼：** https://github.com/import-myself/Membench

---

## 問題與動機

現有 LLM agent 記憶評估框架有三大缺陷：

1. **記憶層次不足**：現有 benchmark（LoCoMo、LongMemEval、PerLTQA）只評估*事實記憶*（Factual Memory，明確陳述的事實），完全忽略*反思記憶*（Reflective Memory，從低層級事實推導出的偏好、摘要等高層級資訊）
2. **互動場景單一**：僅考慮 *participation scenario*（agent 直接第一人稱對話），未涵蓋 *observation scenario*（agent 被動旁觀第三人稱訊息流）
3. **評估指標不完整**：只看準確率，忽略記憶效率（時間成本）與記憶容量（資料量增加後的效能衰退）

---

## 核心貢獻

1. 首個同時涵蓋兩種互動場景 × 兩種記憶層次的資料集
2. **MemBench**：包含四項評估指標的完整 benchmark
3. 在統一框架下評估 7 種記憶機制，使用多種基礎 LLM
4. 公開資料集與程式碼

---

## 兩種互動場景 × 兩種記憶層次

### 互動場景

| 場景 | 說明 |
|------|------|
| **Participation（參與）** | Agent 直接與使用者進行多輪對話（第一人稱） |
| **Observation（旁觀）** | Agent 被動記錄使用者訊息流，不回應（第三人稱） |

### 記憶層次

| 層次 | 說明 | 範例 |
|------|------|------|
| **Factual Memory（事實記憶）** | 對話中明確陳述的事實 | 「我住在北京」 |
| **Reflective Memory（反思記憶）** | 從多個低層級事實推導出的高層級資訊 | 從多次食物偏好推導出「偏好辣味」 |

---

## 資料集情境

MemBench 以**個人助理 agent 服務單一使用者**為核心情境，並延伸出兩種互動模式：

### Participation（參與）情境
Agent 扮演使用者的**個人助理**，與使用者進行多輪日常對話。使用者在對話中自然流露個人資訊，例如：
- 提及家人、同事（「我媽媽下週來找我」）
- 分享生活事件（「昨天去了一家新餐廳」）
- 表達偏好（「我最近很喜歡看科幻片」）

Agent 需要記住這些資訊，以便在後續對話中提供個人化服務。

### Observation（旁觀）情境
Agent 作為**後台記錄系統**，被動接收使用者的訊息串流（類似監看使用者的聊天記錄或社群動態），但不直接參與對話。訊息密度較低，內容較為簡短直接。

### 記憶內容涵蓋
- **事實類**：姓名、年齡、生日、居住地、職業、重要事件時間等
- **反思類**：從多次提及的電影、食物、書籍歸納出偏好風格（例如偏好動作片、喜歡辣食）；從連續情緒表達歸納出情緒傾向

> **一句話情境**：個人助理 agent 在與 500 位虛擬使用者的長期對話（或被動旁觀）中，累積並管理每位使用者的個人資訊與偏好側寫。

---

## 資料集建構

### 建構流程（基於 MemSim 框架）

1. **使用者關係圖取樣**：為 500 位合成使用者建立包含個人資料、事件、地點、物品的關係圖
   - 低層級屬性（factual）：年齡、生日、地點、事件時間等
   - 高層級屬性（reflective）：電影偏好、食物口味偏好、書籍偏好——從三個真實推薦資料集（MovieLens, Food, Goodreads）取最常被正評的類別

2. **記憶資料集建構**：
   - Observation：直接生成使用者訊息流
   - Participation：「自我對話」方法，將偏好資訊自然嵌入多輪對話

### 資料規模

| 資料類型 | Sessions | Questions | 平均 Tokens/軌跡 |
|----------|----------|-----------|----------------|
| PS-RM（參與 × 反思） | 3,500 | 3,500 | 2,195 |
| PS-FM（參與 × 事實） | 51,000 | 39,000 | 10,285 |
| OS-RM（旁觀 × 反思） | 2,000 | 2,000 | 745 |
| OS-FM（旁觀 × 事實） | 8,500 | 8,500 | 617 |

### 評估子資料集
- **Sub-dataset 1**：~10k tokens/session（每個場景各數百題）
- **Sub-dataset 2**：~100k tokens/session（加入 Twitter 新聞噪音 session 填充至 100k+）

---

## 四項評估指標

| 指標 | 說明 |
|------|------|
| **Memory Accuracy** | 多選題正確率（避免開放式回答的評分不穩定） |
| **Memory Recall（Recall@10）** | 取回式機制是否能在 Top-10 中找到關鍵證據輪次 |
| **Memory Capacity** | 記憶量增加時準確率的衰退曲線（找出容量上限） |
| **Memory Efficiency** | 每次操作的讀取時間（RT）與寫入時間（WT） |

---

## 評估的事實記憶子類型（共 8 種）

| 類型 | 難度 |
|------|------|
| Single-hop | 易 |
| Multi-hop | 易～中 |
| Comparative | 難 |
| Aggregative | 難 |
| Post-processing | 中 |
| Knowledge-updating | 中 |
| Single-session-assistant | 中 |
| Multi-session-assistant | 中 |

---

## 主要實驗結果

### 事實記憶（Factual Memory）準確率

**Sub-dataset 1（10k tokens）：**
- 簡單機制（FullMemory 0.647、RetrievalMemory 0.692）優於複雜機制
- Observation 場景普遍比 Participation 更高（RetrievalMemory 達 0.883）
- 複雜機制（MemGPT 0.455、MemoryBank 0.442、SCMemory 0.355）表現差

**Sub-dataset 2（100k tokens）：**
- **RetrievalMemory 是唯一能擴展的機制**（0.833/0.933）
- FullMemory 跌至 0.489；RecentMemory 跌至 0.422（超出 context window）
- 複雜機制維持在 0.41–0.46，無明顯改善

### 反思記憶（Reflective Memory）準確率

**Sub-dataset 1：**
- GenerativeAgent（0.742）、MemGPT（0.733）、FullMemory（0.733）表現較佳
- MemoryBank 在 Observation 場景最強（0.900）

**Sub-dataset 2：**
- 大多數機制崩潰（GenerativeAgent 0.333、MemGPT 0.367）
- RetrievalMemory 依然維持高效能（0.833/0.933）
- 崩潰原因：context window 限制 + 遺忘機制將反思內容丟棄

### 效率比較

| 機制 | 寫入時間 | 讀取時間 |
|------|----------|----------|
| FullMemory / RecentMemory | 幾乎即時 | 幾乎即時 |
| MemoryBank | 8–18 秒/次（最高） | 低 |
| MemGPT | 中等 | ~4.5 秒/次（最高） |

### 基礎 LLM 比較

- **GPT-4o-mini** 在事實記憶最強（FullMemory: 0.736/0.864）
- **Meta-Llama-3.1-8B** 在事實記憶弱（0.519）但反思記憶尚可
- 相同記憶機制下，基礎 LLM 的選擇對效能影響顯著

---

## 結論與啟示

1. **取回式記憶（Retrieval-based）是唯一在大規模下穩健的機制**——10k 和 100k tokens 均維持高效能
2. **複雜設計的機制（GenerativeAgent、MemoryBank、MemGPT）不可靠**——在更嚴格的評估下往往不如簡單基線，可能存在設計缺陷
3. **反思記憶在短 context 下可達成**，但隨互動歷史增長難以維持——尚待研究
4. **效率與容量的取捨不可忽視**：高讀寫成本在實際部署中可能不可接受

### 局限
- 資料集目前聚焦結構化資料（實體屬性），尚未涵蓋非結構化記憶
- 僅評估記憶模組本身，非端對端 agent pipeline

---

## 與 MemoryAgentBench 的比較

| 面向 | MemBench | MemoryAgentBench |
|------|----------|-----------------|
| 記憶層次 | Factual + **Reflective** | Factual only |
| 互動場景 | Participation + **Observation** | Participation only |
| 評估指標 | Accuracy + Recall + **Capacity + Efficiency** | Accuracy |
| Context 規模 | 最大 ~100k tokens | 最大 1.44M tokens |
| 記憶能力分類 | 無明確分類 | AR / TTL / LRU / **SF（選擇性遺忘）** |
| 評估架構數量 | 7 種機制 | 20+ 種 agent |
