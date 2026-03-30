# 論文筆記：BEAM

**標題：** BEYOND A MILLION TOKENS: BENCHMARKING AND ENHANCING LONG-TERM MEMORY IN LLMS
**作者：** Mohammad Tavakoli, Alireza Salemi, Carrie Ye, Mohamed Abdalla, Hamed Zamani, J. Ross Mitchell
**發表：** 2025 年 10 月提交，2026 年 2 月修訂 | arXiv:2510.27246
**論文連結：** https://arxiv.org/pdf/2510.27246
**程式碼：** https://github.com/mohammadtavakoli78/BEAM

---

## 問題與動機

LLM 被廣泛部署於長對話、RAG、長文件分析、法律研究等場景，但現有長期記憶 benchmark 有三大缺陷：

1. **缺乏敘事連貫性**：多數 benchmark 直接串接不同 session，話題突然切換，使得片段過於容易分離，降低了真正長程推理的難度
2. **領域覆蓋窄**：現有資料集多侷限於個人生活場景
3. **能力測試膚淺**：偏重簡單回憶，忽略矛盾解消、資訊演變追蹤、指令持續遵循等關鍵能力

此外，現有任何 benchmark 均無法擴展至超過 1M tokens，而現代 LLM 的 context window 已達 1M tokens，需要更大規模的評估。

---

## 核心貢獻

1. **BEAM（Benchmark）**：自動化 pipeline 生成連貫、多領域的單用戶單 agent 對話，支援 100K 至 **10M tokens**，附 2,000 道人工驗證探測題，涵蓋 10 種記憶能力
2. **LIGHT（Framework）**：認知科學啟發的三記憶系統框架，在最強 baseline 上平均提升 3.5%–12.69%（特定配置最高 +155.7%）
3. **三個新增記憶維度**：Instruction Following、Event Ordering、Contradiction Resolution（先前 benchmark 均未涵蓋）

---

## 資料集情境

**參與者**：
- 單一使用者（具備生成屬性：姓名、年齡、性別、地點、職業、MBTI 人格，及連結家人、朋友、熟人的關係圖）
- AI assistant

**對話性質**：
- **雙向多輪對話**，包含追問與澄清，非單純問答序列
- **跨輪次維持敘事連貫性**（而非拼接獨立 sessions）
- 內容橫跨多領域：程式開發、個人生活、職業情境等
- 對話有明確時間軸，事件有先後順序

**人工評分（1–5）**：連貫性 4.53、真實感 4.57、複雜度 4.64

> **一句話情境**：一位有豐富背景的使用者與 AI 助理進行跨越數月、涵蓋工作與生活多面向的長期連貫對話，對話長度可達 10M tokens。

---

## 資料集建構

### 五階段 Pipeline

**Stage 1 — 對話計畫生成**
- 從使用者屬性、領域、目標能力生成高層敘事骨架（N 個子計畫，各 M 個重點）
- ≤1M tokens：生成單一計畫
- 10M tokens：生成十個互扣計畫（Sequential Expansion 或 Hierarchical Decomposition）

**Stage 2 — 使用者話語生成**
- 子計畫切成 K 個批次，LLaMA-3.3 70B 每批生成 I 個使用者問句

**Stage 3 — Assistant 話語生成**
- 一個 LLM 扮演 assistant，另一個扮演使用者
- 問題偵測模組（閾值 δ₁=2）+ 追問偵測模組（δ₂=2）驅動迭代澄清循環

**Stage 4 — 探測題生成**
- GPT-4.1-mini 為每種能力選取候選重點，連結到對話輪次
- 人工評審驗證並修正

**Stage 5 — 驗證與 Nugget 建立**
- 無效題目丟棄；每種能力每組對話選 2 題（共 20 題/組）
- 從理想答案拆解出原子評估標準（nuggets）

### 資料集規模

| 指標 | 數值 |
|------|------|
| 對話組數 | 100 |
| 長度梯度 | 100K / 500K / 1M / 10M tokens |
| 每組探測題數 | 20（每種能力 2 題） |
| 總探測題數 | 2,000 |
| 記憶能力數 | 10 |

---

## 十種記憶能力

| 能力 | 說明 | 是否為新增 |
|------|------|----------|
| **Abstention** | 缺乏證據時正確拒答 | |
| **Contradiction Resolution** | 偵測並調和對話中的矛盾陳述 | ★ 新 |
| **Event Ordering** | 識別並重建資訊的演變順序 | ★ 新 |
| **Information Extraction** | 精確回憶實體與事實細節 | |
| **Instruction Following** | 在長對話中持續遵守使用者指定的限制條件 | ★ 新 |
| **Knowledge Update** | 隨新事實出現修訂已儲存的資訊 | |
| **Multi-Hop Reasoning** | 整合多個不相鄰片段進行推理 | |
| **Preference Following** | 根據演變中的偏好給出個人化回應 | |
| **Summarization** | 抽象壓縮對話內容 | |
| **Temporal Reasoning** | 推理明確與隱含的時間關係 | |

---

## 評估指標

- **Nugget-based 評分**（9 種能力）：LLM judge 對每個 nugget 給 0 / 0.5 / 1 分，取平均
- **Kendall tau-b 係數**（Event Ordering）：LLM 等價偵測器對齊生成事件與 nuggets，tau-b 同時捕捉排序與存在性

---

## 主要實驗結果

### 整體準確率（各長度）

**100K tokens：**

| 模型 | Vanilla | RAG | LIGHT |
|------|---------|-----|-------|
| Qwen 2.5 | 0.280 | 0.269 | **0.311** |
| LLaMA 3.3 | 0.240 | 0.323 | **0.358** |
| Gemini | 0.242 | 0.280 | **0.294** |
| GPT-4.1 | 0.239 | 0.309 | **0.345** |

**1M tokens：**

| 模型 | Vanilla | RAG | LIGHT |
|------|---------|-----|-------|
| Qwen 2.5 | 0.193 | 0.285 | **0.309** |
| LLaMA 3.3 | 0.259 | 0.307 | **0.336** |
| Gemini | 0.199 | 0.271 | **0.284** |
| GPT-4.1 | 0.191 | 0.302 | **0.336** |

**10M tokens：**

| 模型 | Vanilla | RAG | LIGHT |
|------|---------|-----|-------|
| Qwen 2.5 | 0.133 | 0.211 | **0.238** |
| LLaMA 3.3 | 0.104 | 0.249 | **0.266** |
| Gemini | 0.122 | 0.216 | 0.192（RAG 較優） |
| GPT-4.1 | 0.109 | 0.218 | **0.226** |

### LIGHT vs. Vanilla 提升幅度

| Context 長度 | 最大提升（模型） |
|-------------|----------------|
| 100K | +49.1%（LLaMA）、+44.3%（GPT-4.1） |
| 1M | +75.9%（GPT-4.1）、+60.1%（Qwen） |
| 10M | **+155.7%（LLaMA）**、+107.3%（GPT-4.1） |

### 各能力最大提升（LIGHT vs. Vanilla）

| 能力 | 最大提升 |
|------|----------|
| Summarization | +160.6% |
| Preference Following | +76.5% |
| Information Extraction | +56.7% |
| Temporal Reasoning | +56.3% |
| Instruction Following | +39.5% |
| Multi-Hop Reasoning | +27.2% |
| **Contradiction Resolution** | 全部方法均極低，是最大未解挑戰 |

### 消融實驗（10M tokens）

| 移除元件 | 效能下降 |
|----------|----------|
| Vector Retrieval（情節記憶） | -8.5% |
| Noise Filtering（噪音過濾） | -8.3% |
| Working Memory | -5.7% |
| Scratchpad | -3.7% |

最佳取回數 k=15；k=20 因噪音略降，k=10 在 10M 下有害。

---

## LIGHT 框架架構

LIGHT 結合三種平行記憶系統，靈感來自認知科學：

### 元件一：情節長期記憶（向量取回）
- 每輪對話後，Qwen2.5-32B-AWQ 提取鍵值對（實體 → 屬性）與摘要
- 以 BAAI/bge-small-en-v1.5 嵌入，存入 FAISS 向量資料庫
- 推理時取 Top-k=15 最近鄰的原始對話片段

### 元件二：工作記憶（短期緩衝）
- 直接傳入最近 z 輪對話，捕捉即時脈絡

### 元件三：便條紙（Scratchpad）
- 每輪提取顯著內容（語意知識、自傳細節、前瞻性記憶、脈絡元資料），迭代合併
- 超過 30K tokens 時由 GPT-4.1-nano 壓縮至 15K tokens
- 推理時用 SemanticChunker 分塊，再以 Qwen2.5-32B-AWQ 做二元相關性過濾

**推理**：三個元件並行運作，輸出合併後送入 backbone LLM

---

## 結論與啟示

1. **BEAM 是首個達到 10M tokens 且保持敘事連貫的 benchmark**，現有 LLM 對話長度越長效能越差
2. **LIGHT 在所有模型、所有長度下持續優於 Vanilla 和 RAG**，三元件在長 context 下互補效果更明顯
3. **Contradiction Resolution 是所有方法的死角**，是最迫切的未解挑戰
4. 認知科學三記憶架構（情節 + 工作 + 便條紙）驗證了心理學原則在 LLM 記憶設計中的有效性

---

## 與其他 Benchmark 的關係

| 面向 | BEAM | LongMemEval | MemoryAgentBench | LoCoMo |
|------|------|-------------|-----------------|--------|
| 最大 Context | **10M tokens** | 1.5M tokens | 1.44M tokens | ~9K tokens |
| 敘事連貫性 | ✓（單一連貫對話） | 部分（多 session 串接） | ✗（資料集彙整） | ✓ |
| 題目數 | 2,000 | 500 | 2,071 | 7,512 |
| Contradiction Resolution | ✓（新增） | ✗ | ✓（FactConsolidation） | ✗ |
| Instruction Following | ✓（新增） | ✗ | ✗ | ✗ |
| 提出解法 | ✓（LIGHT） | ✓（四個 CP 優化） | ✗ | ✗ |
