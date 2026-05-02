# Self-Consistency Improves Chain of Thought Reasoning in Language Models

## 基本資訊

| 項目 | 內容 |
|------|------|
| **標題** | Self-Consistency Improves Chain of Thought Reasoning in Language Models |
| **作者** | Wang, Xuezhi; Wei, Jason; Schuurmans, Dale; Le, Quoc; Chi, Ed; et al. |
| **年份** | 2022 |
| **arXiv ID** | 2203.11171 |
| **發布機構** | Google Brain / Google Research |
| **原文連結** | https://arxiv.org/abs/2203.11171 |

---

## 研究背景與動機

Chain-of-Thought (CoT) prompting 已被證明能有效提升大型語言模型在複雜推理任務上的表現。然而，CoT 存在一個關鍵問題：

**傳統 CoT 使用 greedy decoding（貪心解碼）**
- 只選擇每個步驟機率最高的下一個 token
- 單一路徑推理，缺乏多樣性
- 可能導致錯誤累積

Self-Consistency 正是為了解決這個問題而被提出。

---

## 核心方法 (Methodology)

### 核心理念
**"多數投票" 機制應用於語言模型推理**

```
1. 採樣多個不同的推理路徑（而非只貪心選擇）
2. 每個路徑產生一個最終答案
3. 選擇被最多推理路徑支持的答案（majority vote）
```

### 與 Greedy CoT 的對比

| 方面 | Greedy CoT | Self-Consistency |
|------|------------|------------------|
| 解碼策略 | 貪心選擇 | 多路徑採樣 |
| 推理路徑 | 單一 | 多樣 |
| 錯誤處理 | 錯誤累積 | 錯誤稀釋 |
| 計算成本 | 低 | 較高 |

### 實現細節

```python
# 概念性算法
responses = model.sample(                    # 1. 採樣多個回應
    prompt + question, 
    num_samples=40
)

answers = [extract_answer(r) for r in responses]  # 2. 提取答案

final_answer = majority_vote(answers)         # 3. 多數投票
```

### 溫度參數
- 使用較高的 temperature（如 0.7）促進多樣性
- 確保採樣的推理路徑具有足夠差異性

---

## 核心貢獻 (Main Findings)

### 1. 性能提升效果
在多個推理基準測試上的提升：

| 任務 | Greedy CoT | Self-Consistency | 提升 |
|------|------------|------------------|------|
| GSM8K (數學) | 46.9% | 74.4% | +27.5% |
| SVAMP (數學) | 57.4% | 76.4% | +19.0% |
| StrategyQA | 72.4% | 83.4% | +11.0% |
| ARC-Challenge | 72.4% | 81.4% | +9.0% |
| ANLI (自然語言推理) | 36.4% | 54.8% | +18.4% |

### 2. 對錯誤的容忍度
- 單一路徑推理容易一路錯到底
- 多路徑採樣讓正確路徑有更大機會被選擇
- 錯誤推理路徑會被「投票」機制稀釋

### 3. 模型規模效益
Self-Consistency 在大型模型（如 PaLM-540B）上效果更顯著：
- 小模型：提升有限
- 中型模型：中等提升
- 大型模型：顯著提升

### 4. 無需額外訓練
- 不需要訓練或微調
- 完全基於 prompting 技術
- 適用於任何預訓練語言模型

### 5. 對 Prompt 變化的魯棒性
- 對不同 CoT prompt 的變化較不敏感
- 提升了推理的穩定性

---

## 為什麼重要 (Why It Matters)

1. **簡單而有效**：幾乎不增加實現複雜度
2. **通用性**：適用於各種類型的推理任務
3. **性能提升顯著**：在多個基準上實現 SOTA 或接近 SOTA
4. **無需訓練**：節省大量計算資源和時間
5. **理論直覺**：利用語言模型分佈的「自我一致性」

---

## 實際應用場景

1. **數學問題求解**
   - 學生作業輔助
   - 工程計算驗證

2. **程式碼生成與除錯**
   - 多路徑分析程式邏輯
   - 錯誤檢測與修正

3. **醫療診斷輔助**
   - 多種症狀分析
   - 鑑別診斷

4. **法律推理**
   - 多角度案件分析
   - 判例研究

5. **複雜決策支援**
   - 需要多種推理角度的商業決策
   - 風險評估

---

## 局限性 (Limitations)

1. **計算成本增加**：需要採樣多個路徑，延遲增加
2. **簡單任務不需要**：對於簡單任務可能過度複雜
3. **資源消耗**：40 次採樣需要更多計算資源
4. **答案提取依賴**：需要可靠的答案提取方法

---

## 與其他技術的結合

- **+ Chain-of-Thought**: 基礎框架
- **+ Tree of Thoughts**: 可進一步擴展為樹狀搜索
- **+ Self-Ask**: 增強問題分解能力
- **+ Least-to-Most**: 複雜問題分解

---

## 摘要

Self-Consistency 是一種簡單而有效的語言模型推理改進方法。通過採樣多條推理路徑並選擇多數投票結果，它能顯著提升 CoT 推理的準確性。實驗表明，這種方法在多個推理基準測試上都能帶來實質性的性能提升，特別是在大型語言模型上效果更佳。作為一種不需要額外訓練的 prompting 技術，它為提升 LLM 推理能力提供了一個簡單有效的方向。

---

*筆記整理日期：2026年4月17日*
