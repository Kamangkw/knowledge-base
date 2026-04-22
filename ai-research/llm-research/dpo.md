# Direct Preference Optimization (DPO): 直接偏好優化

## 基本資訊

| 項目 | 內容 |
|------|------|
| **標題** | Direct Preference Optimization: Your Language Model is Secretly a Reward Model |
| **作者** | Rafael Rafailov, Archit Sharma, Eric Mitchell, Stefano Ermon, Christopher D. Manning |
| **年份** | 2023 |
| **arXiv ID** | 2305.18290 |
| **發布機構** | Stanford University |
| **原文連結** | https://arxiv.org/abs/2305.18290 |

---

## 研究背景與動機

大型語言模型通過無監督預訓練學習了廣泛的知識和推理技能，但要精確控制其行為卻非常困難。現有對齊方法需要：

### 傳統 RLHF 流程

```
1. 收集人類偏好標註（人類評估回應質量）
2. 訓練 Reward Model（獎勵模型）：學習人類偏好
3. PPO 強化學習：用 Reward Model 信號 fine-tune LLM
4. 約束：確保模型不要太偏離原始預訓練
```

**問題所在**：
- 複雜且不穩定：三階段 pipeline，任何一步出錯都會影響最終效果
- 需要大量人類標註：昂貴且難以規模化
- 訓練不穩定：PPO 需要大量超參數調優
- 內存密集：需要同時運行 policy model、reference model、reward model、value network

**核心願景**：能否用一個簡單的分類損失，直接將 LLM 對齊到人類偏好，而不需要強化學習？

---

## 核心方法 (Methodology)

### DPO 的核心洞察

作者發現，RLHF 中的 reward model 可以被重新參數化，使得**最優策略**可以用**閉式解**表達。

```
RLHF 問題的本質：
  max E[r(x, y)] - β * KL(πθ(y|x) || πref(y|x))
  
  其中 r(x, y) 是 reward function

DPO 的洞察：
  → 最優 policy π* 可以直接從 reward model 推導
  → 不需要明確的 reward model 或 PPO
```

### DPO 損失函數

```python
# DPO 損失函數（概念性）
loss = - E_{(x, y_w, y_l) ~ D}[
    log σ(β * (log πθ(y_w|x) - log πref(y_w|x) 
              - log πθ(y_l|x) + log πref(y_l|x)))
]
```

其中：
- `y_w` = 人類偏好的回應（chosen）
- `y_l` = 人類不偏好的回應（rejected）
- `πθ` = 當前策略模型
- `πref` = 原始 reference 模型
- `β` = 控制對齊強度的參數（通常 0.1-0.3）

### 與 RLHF 的對比

| 方面 | RLHF | DPO |
|------|------|-----|
| 訓練階段 | 3步（Reward Model → PPO） | **1步** |
| 需要採樣 | 是（PPO 需要） | **否** |
| 超參數調優 | 大量（PPO） | **極少** |
| 穩定性 | 複雜、不穩定 | **簡單、穩定** |
| 內存需求 | 高（4個模型） | **低（2個模型）** |
| 實現複雜度 | 高 | **低** |

### 訓練流程

```
┌────────────────────────────────────────────────────────┐
│  DPO 訓練流程                                          │
├────────────────────────────────────────────────────────┤
│                                                        │
│  1. 準備偏好數據對 (x, y_w, y_l)                       │
│     - y_w: 人類偏好的回應                              │
│     - y_l: 人類不偏好的回應                            │
│                                                        │
│  2. 計算 Log Ratio                                     │
│     - log πθ(y|x) / πref(y|x)                        │
│     - 衡量當前模型與原始模型的偏離程度                  │
│                                                        │
│  3. 應用 DPO 損失                                       │
│     - 直接最大化偏好回應的相對概率                      │
│     - 同時保持與 reference model 的相似度               │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## 核心貢獻 (Main Findings)

### 1. 情感控制能力

在情感控制任務上，DPO 超越 PPO-based RLHF：

| 模型 | 控制情感能力 | 質量評估 |
|------|------------|---------|
| RLHF (PPO) | ✓ 有效 | 中等 |
| **DPO** | ✓✓ 更強 | 更高 |

### 2. 摘要任務

在 TL;DR 摘要任務上：
- DPO 達到與 RLHF 相當的質量
- 實現更簡單，訓練更穩定

### 3. 單輪對話

在單輪對話任務上：
- DPO 匹配或超越 RLHF 的回應質量
- 大幅降低實現複雜度

### 4. 實驗穩定性

- 不需要調整 PPO 相關超參數
- 訓練收斂更可預測
- 降低硬件需求（內存）

---

## 為什麼重要 (Why It Matters)

1. **民主化對齊技術**：讓更多研究者和公司能夠實現 LLM 對齊
2. **理論優雅**：將複雜的 RL 問題簡化為簡單的分類問題
3. **實際價值**：
   - 訓練速度更快
   - 實現更簡單
   - 資源需求更低
4. **成為主流**：DPO 及其變體（如 ORPO、IPO）已成為 RLHF 的主要替代方案
5. **穩定性**：解決了 RLHF 訓練不穩定的核心痛點

---

## 實際應用場景

1. **開源模型對齊**
   - Llama、Mistral 等模型的 RLHF/DPO fine-tuning
   - 成為開源社區的標準對齊方法

2. **企業定製化**
   - 根據特定領域調整 LLM 行為
   - 客服、醫療、法律等專業領域

3. **安全性增強**
   - 減少有害輸出
   - 提高回答質量

4. **研究加速**
   - 快速迭代對齊實驗
   - 降低對齊研究的門檻

---

## 局限性 (Limitations)

1. **偏好數據依賴**：仍需要高質量的偏好標註數據
2. **理論保證**：閉式解的推導依賴某些假設
3. **超參數 β**：需要調整對齊強度參數
4. **Reward Model 的角色**：DPO 隱式學習 reward，可能不如顯式 reward model 靈活

---

## 相關研究

- **RLHF**：傳統的三階段對齊方法
- **PPO**：用於 RLHF 的强化學習算法
- **Constitutional AI**：Anthropic 的對齊方法
- **IPO** (Iterative Preference Optimization)：DPO 的變體
- **ORPO** (Odds Ratio Preference Optimization)：無需 reference model 的方法

---

## 摘要

Direct Preference Optimization (DPO) 是一項革命性的技術，它將 RLHF 的三階段流程簡化為一個端到端的分類損失。通過重新參數化 reward model，DPO 使得 LLM 對齊可以在不需要強化學習的情況下完成。這個方法既簡單又穩定，已成為開源模型對齊的事實標準，大幅降低了高質量 LLM 的訓練門檻。

---

*筆記整理日期：2026年4月22日*
