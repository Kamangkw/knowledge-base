---
title: Chain-of-Thought Prompting: 思維鏈提示
date: 2026-04-22
source: 自行研究整理（主要參考：Wei J et al., "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models" arXiv:2201.11903）
tags: [Prompt Engineering, Chain-of-Thought, 推理, LLM]
description: 介紹Chain-of-Thought Prompting技術，讓大型語言模型展示推理過程，提升複雜問題的解答能力。
---

# Chain-of-Thought Prompting: 思維鏈提示

## 基本資訊

| 項目 | 內容 |
|------|------|
| **標題** | Chain-of-Thought Prompting Elicits Reasoning in Large Language Models |
| **作者** | Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Brian Ichter |
| **年份** | 2022 |
| **arXiv ID** | 2201.11903 |
| **發布機構** | Google Brain / Google Research |
| **原文連結** | https://arxiv.org/abs/2201.11903 |

---

## 研究背景與動機

大型語言模型（LLM）在自然語言任務上表現出色，但在複雜推理任務上仍有明顯不足。傳統的 prompting 方法（如 few-shot prompting）只能提供輸入-輸出對，模型難以展示推理過程。

**核心問題**：如何激發 LLM 的複雜推理能力？

---

## 核心方法 (Methodology)

### 思維鏈（Chain of Thought）是什麼？

Chain-of-Thought（CoT）是一種**系列的中間推理步驟**，它不是直接給出答案，而是讓模型先「思考」：

```
傳統 Prompting:
  輸入：問題
  輸出：答案 ✓或✗

Chain-of-Thought Prompting:
  輸入：問題 + 推理範例
  輸出：推理過程 → 答案 ✓✓✓
```

### 關鍵要素

1. **Few-shot CoT 範例**
   - 在 prompt 中提供幾個包含推理過程的範例
   - 每個範例包含：中間推理步驟 + 最終答案

2. **中間步驟**
   - 將複雜問題分解為多個簡單步驟
   - 每個步驟都是可解釋的推理過程

3. **觸發時機**
   - CoT 主要在**足夠大的模型**（≥100B 參數）上展現效果
   - 小模型可能學到「表面模式」而非真正推理

### Prompt 格式示例

```
問題：花園裡有 5 朵花，每朵有 3 片花瓣。總共有多少花瓣？

傳統答案：15 片

CoT 答案：
- 花園裡有 5 朵花
- 每朵有 3 片花瓣
- 所以總共有 5 × 3 = 15 片花瓣
- 答案是 15 片花瓣
```

---

## 核心貢獻 (Main Findings)

### 1. 數學推理大幅提升

| 任務 | Without CoT | With CoT | 提升幅度 |
|------|------------|----------|---------|
| GSM8K (數學應用題) | 17.9% | **67.0%** | +49.1% |
| SVAMP (數學變體) | 40.8% | **66.1%** | +25.3% |
| MAWPS (數學) | 42.7% | **66.9%** | +24.2% |

### 2. 需要「規模」才能浮現

CoT 的效果與模型規模密切相關：

| 模型規模 | Without CoT | With CoT |
|----------|------------|----------|
| PaLM 8B | 1.5% | 1.6% |
| PaLM 62B | 4.3% | 13.5% |
| PaLM 540B | 33.6% | **67.0%** |

> 結論：小模型（<100B）從 CoT 獲益甚微；CoT 能力需要足夠大的模型才會「湧現」。

### 3. 適用任務類型

- **數學推理**：算術應用題、幾何證明
- **常識推理**：多跳問答、因果推斷
- **符號推理**：序列操作、邏輯推導

### 4. 與其他技術相容

CoT 可以與其他 prompting 技術結合：
- **+ Self-Consistency**：多路徑採樣 + 多數投票
- **+ Tree of Thoughts**：將線性推理擴展為樹狀搜索
- **+ Zero-shot CoT**：自動生成推理步驟（無需範例）

---

## Zero-shot Chain-of-Thought

後續研究發現，僅需在 prompt 末尾加上「Let's think step by step」，無需提供範例，模型也會自動生成推理步驟。這就是 **Zero-shot CoT**。

```
問題：[數學問題]
回答：讓我一步一步地思考...
[自動展開推理過程]
```

---

## 為什麼重要 (Why It Matters)

1. **開啟推理研究新方向**：首次展示 LLM 具備隱式推理能力
2. **可解釋性**：推理過程透明化，人類可檢視思維路徑
3. **零訓練成本**：不需要 fine-tuning，只需改變 prompt 格式
4. **為後續研究奠定基礎**：Self-Consistency、Tree of Thoughts 等都基於 CoT
5. **實用價值**：直接提升現有 LLM 系統的推理能力

---

## 局限性 (Limitations)

1. **規模門檻**：小模型無法從 CoT 獲益
2. **推理質量不穩定**：推理步驟可能包含錯誤或繞路
3. **難以驗證**：模型可能生成看似合理但實際錯誤的推理
4. **評估困難**：需要人工或自動評估推理過程的正確性

---

## 相關資源

- 論文：https://arxiv.org/abs/2201.11903
- 相關：Self-Consistency (Wang et al., 2022)
- 相關：Tree of Thoughts (Yao et al., 2023)

---

## 摘要

Chain-of-Thought Prompting 是一種簡單但強大的技術，通過在 prompt 中提供包含中間推理步驟的範例，顯著提升 LLM 在數學、常識和符號推理任務上的表現。這種能力主要在大型模型（≥100B）上湧現，不需要任何訓練或 fine-tuning。CoT 為 LLM 推理研究開闢了新方向，並成為後續多種推理增強技術的基礎。

---

*筆記整理日期：2026年4月22日*
