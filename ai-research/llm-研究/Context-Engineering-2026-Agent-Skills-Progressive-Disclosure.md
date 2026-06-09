---
title: Context Engineering 2026：Agent Skills 與 Progressive Disclosure 模式
date: 2026-06-10
source: 自行研究整理（Toward AI、BeamSec、Frontiers in AI）
tags: [LLM, AI-Agent, Context-Engineering, Agent-Skills, Progressive-Disclosure]
description: Context Engineering 已成為 2026 年 AI Engineering 的核心學科，探討有限注意力預算下，如何以最小 tokens 最大化 LLM 輸出品質
---

# Context Engineering 2026：Agent Skills 與 Progressive Disclosure 模式

## 基本資訊
- **起源**：2024-2025 年隨著 LLM 上下文窗口快速擴展而興起
- **關鍵人物/機構**：Anthropic、OpenAI、Google、GitHub、Cursor
- **核心問題**：LLM 的注意力預算有限，上下文越長，精度越低，推理越弱
- **核心原則**：以最小的高信號 tokens 集合，最大化達成目標結果的概率

## 為何重要

Context Engineering 已經從一個邊緣關注點，演化為 AI Engineering 的核心學科。隨著企業大規模部署 AI Agent，上下文管理失效成為 production 失敗的首要原因之一。

**注意力預算競爭**：進入上下文窗口的所有元素都在競爭注意力——系統指令、工具定義、MCP 資源、檢索文檔、對話歷史、累積操作歷史，全部同時競爭有限的注意力資源。

**兩個已知失效模式**：
- **Lost in the Middle**：重要資訊位於長上下文的中間位置時，模型完全忽略
- **Needle in a Haystack**：模型難以在龐大文檔中找到特定資訊

## 核心內容

### 模式一：Progressive Disclosure 與 Agent Skills

#### 問題背景
一個處理多個領域的 Agent（客戶支援、帳單、退款、引導入職）浪費大量上下文在不相關的指引上。如果分別啟動獨立子 Agent，會增加編排複雜度、邏輯重複、延遲上升。

#### 解決方案：分層載入資訊

**第一層：Discovery（發現層）**
- 只載入的名稱和描述
- 中位元 tokens：~80 tokens
- 作用：讓 Agent 知道有哪些技能可用

**第二層：Activation（激活層）**
- 當技能相關時，載入完整指令
- 範圍：275 到 8,000 tokens
- 作用：提供執行該技能所需的所有上下文

**第三層：Execution（執行層）**
- 只在執行任務期間，載入腳本和參考資料
- 作用：最小化上下文佔用的峰值

#### Agent Skills 格式

Anthropic 在 2025 年 12 月發佈的標準格式：
- Markdown 檔案 + YAML frontmatter
- 包含技能名稱、觸發條件、執行指令、行為約束
- OpenAI、Google、GitHub、Cursor 在數週內全部採用

**YAML frontmatter 範例結構**：
```yaml
---
name: skill-name
trigger: when-this-situation-arises
instructions: |
  Full detailed instructions here
constraints:
  - never do X
  - always do Y
---
```

#### 單一 Agent 身份切換

傳統做法：為每個技能啟動獨立子 Agent（如「PDF Agent」、「試算表 Agent」）

**2026 年最佳實踐**：單一 Agent 按需切換身份
- **At Rest（閒置時）**：保持 base identity
- **During Task（任務中）**：採納技能指令、約束、語氣和行為模式
- **After Task（任務後）**：返回 base identity

Claude Code 的工作模式就是如此——它不會啟動獨立的「PDF Agent」和「試算表 Agent」，而是一個 Agent 激活相關技能，切換身份來完成任務。

#### Agent 自我創作 Skills

Claude Code 支援透過 skill-creator skill 讓 Agent 自我創作新技能：
- Agent 觀察成功的行為模式
- 將其推廣為通用的、可複用的技能
- 使其在未來會話中可用

**分工**：人類撰寫初始技能，Agent 擴展技能庫。

#### Progressive Disclosure 的權衡

| 維度 | 評估 |
|------|------|
| **準確度** | 小型技能集合（<20個）時高；100+個時因描述重疊而下降 |
| **延遲** | 低——Discovery 資料預載入，Activation 只是檔案讀取 |
| **Token 成本** | 閒置時極低（17 個 Anthropic 技能 Discovery 總計 ~1,700 tokens）|
| **可維護性** | 小規模容易；50+ 個技能時複雜度上升 |

### 模式二：上下文召回問題與緩解

#### 「Lost in the Middle」與「Needle in a Haystack」

上下文越長，模型越難識別重要資訊的位置和內容。

**緩解策略**：

**1. Context Compression（語境壓縮）**
- 總結並合併冗餘內容
- 保留核心語義，去除非關鍵細節
- 適用場景：RAG 系統的檢索後壓縮

**2. Retrieval Reranking（檢索重排）**
- 根據滾動相關性信號重新排序檢索結果
- 將最相關的內容放在上下文窗口的「安全位置」（開頭或結尾）
- 適用場景：長文檔問答、多文檔總結

**3. Recency Bias（近因偏差）**
- 模型傾向過度重視上下文末尾的資訊
- 中間位置的資訊被低估
- **Position Averaging**：訓練時對位置進行週期性編碼，減少位置對注意力分佈的影響

**4. Semantic Chunking（語義分塊）**
- 按主題而非固定 token 數量分割文檔
- 保持每個區塊的語義完整性
- 適用場景：長文檔處理、法律/醫學文檔分析

**5. Synthetic Grounding（合成接地）**
- 添加支撐範例或解釋性文本
- 幫助模型更好地理解檢索內容的上下文
- 適用場景：複雜概念解釋、專業術語理解

**6. Hierarchical Summarization（層次化摘要）**
- 多階段總結：先對文檔片段總結，再對摘要們總結
- 逐層向上傳遞相關內容
- 適用場景：千頁文檔分析、程式碼庫理解

### 架構層面的解決方案

**Sparse Attention（稀疏注意力）**
- 不對所有 tokens 平等計算注意力
- 讓模型選擇性關注關鍵 tokens
- 降低計算複雜度的同時保持核心資訊

**Recursive Large Language Models（遞歸語言模型，MIT）**
- 將龐大文檔存儲在 Python 環境中
- 讓 LLM 以程式化方式探索文檔
- 突破固定上下文窗口限制
- 適用場景：長文檔分析、程式碼庫理解、合同審查

## 企業部署現況（2026）

### 關鍵 adoption 數據

| 指標 | 數值 |
|------|------|
| 組織部署 AI Agent 處理多階段工作流 | >50% |
| 跨職能跨團隊運行複雜 Agent 流程 | 16% |
| 報告可衡量經濟回報 | **80%** |

### AI 編碼領域滲透率最高

- **90%** 的組織使用 AI 輔助開發
- **86%** 在生產環境部署 Agent

**各開發階段節省時間**：
- 規劃和 ideation：58%
- 程式碼生成：59%
- 文檔撰寫：59%
- 程式碼審查和測試：59%

**實際案例**：Doctolib 在數小時內替換了遺留測試基礎設施，功能上線速度提升 40%。

### 常見擴展路徑

組織遵循可預測的 Agent 部署路徑：
1. **Coding（編碼）**——起點，證明價值
2. **Research & Reporting（研究報告）**——下一步
3. **Customer Service（客戶服務）**——擴展
4. **Financial Planning（財務規劃）**——深化
5. **Supply Chain（供應鏈）**——全面自動化

> **關鍵洞察**：將 AI Agent 視為基礎設施（infrastructure）而非創新項目，是 2026 年成功的關鍵。

Context Engineering 直接影響 production 中的：
- 對話歷史導致的 context drift
- 多階段工作流中的任務失敗
- 長上下文帶來的召回失效

## AlphaFold 3：生物分子建模的 Context Engineering 典範

### 架構演進

AlphaFold 系列的架構演進是 Context Engineering 的最佳案例：

| 版本 | 架構 | 創新 | 準確度 |
|------|------|------|--------|
| AF1 (2018) | DNN + 共進化耦合 | 預測殘基間距和接觸圖 | CASP13 FM 第一名 |
| AF2 (2020) | Evoformer + Transformer | 端到端深度學習 | 92.4 GDT |
| AF3 (2024) | Pairformer + Atom Transformer | 統一分子類型token化 | 超過 physics-based 方法 50% |

### AlphaFold 3 的 token 創新

AF3 用 **Pairformer** 替換 **Evoformer**：
- 所有分子類型（蛋白質、DNA、RNA、配體、離子、修饰殘基）統一 token 化
- 適用於原子坐標生成的擴散模型
- 在 PoseBusters 基準上比最好物理方法高出 50%

### AlphaFold 資料庫的擴展

| 時間節點 | 結構數量 | 覆蓋範圍 |
|----------|----------|----------|
| 2021 年 | 300,000 | 人類蛋白組 48% |
| 2024 年 | 2.14 億 | 人類蛋白組 76% |

**關鍵突破**：
- 人類蛋白組覆蓋：48% → 76%
- 完全無結構資訊的蛋白：5,027 → 29
- 「黑暗蛋白組」：26% → 10%

## 局限性與挑戰

### Context Engineering 的局限性

1. **資訊丟失**：壓縮和摘要不可避免地丟失細節
2. **技能集合膨脹**：50+ 個技能後描述重疊，準確度下降
3. **動態上下文**：多階段任務中，上下文不斷變化，靜態分層策略可能失效
4. **位置偏差**：即使有 Position Averaging，模型對不同位置資訊的敏感度仍有差異
5. **計算成本**：稀疏注意力和層次化摘要增加推理複雜度

### 當前未解決的問題

- **超長上下文（100K+ tokens）**：仍面臨「lost in the middle」問題
- **跨模態上下文管理**：視覺+文字+音頻的聯合上下文工程仍不成熟
- **自動化 Context Optimization**：缺乏讓 LLM 自動優化自身上下文的成熟方法

## 實際應用場景

**適合使用 Progressive Disclosure + Agent Skills 的場景**：
- 企業內部 AI 助手（多業務領域支援）
- 開發者工具（Claude Code 類產品）
- 客服 Bot（多產品線支援）

**適合使用 Context Compression 的場景**：
- RAG 系統的檢索後處理
- 長文件分析（法律、醫療、金融報告）
- 多文件對比分析

**適合使用 Hierarchical Summarization 的場景**：
- 巨型代碼庫理解
- 千頁研究文獻綜述
- 企業知識庫整合

## 標籤
#LLM #AI-Agent #Context-Engineering #Agent-Skills #Progressive-Disclosure #Procurement #Enterprise-AI #AlphaFold

---
source: [State of Context Engineering in 2026](https://pub.towardsai.net/state-of-context-engineering-in-2026-cf92d010eab1) | [How Enterprises Are Building AI Agents in 2026](https://beamsec.com/how-enterprises-are-building-ai-agents-in-2026-from-pilots-to-production/) | [AlphaFold 3 Impact](https://www.frontiersin.org/journals/artificial-intelligence/articles/10.3389/frai.2026.1739303/full)
