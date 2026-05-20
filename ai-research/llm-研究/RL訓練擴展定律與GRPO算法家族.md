---
title: RL訓練擴展定律：GRPO算法家族與推理模型的新方向
date: 2026-05-20
source: 自行研究整理（Cameron R. Wolfe Ph.D. Substack + Google Research + DeepSeek R1技術報告）
tags: [AI, 強化學習, GRPO, LLM推理, DAPO, GSPO, RLVR]
description: 2025-2026年LLM推理能力訓練的核心技術——從PPO到GRPO的演進，以及GSPO、DAPO、Dr.GRPO等變種如何解決訓練穩定性、獎勵噪聲、熵崩潰等實際問題。
---

# RL訓練擴展定律：GRPO算法家族與推理模型的新方向

## 概述

2025年被業界稱為「推理模型元年」——OpenAI o系列、DeepSeek R1等模型的崛起，背後的核心技術是**強化學習與可驗證獎勵（RLVR）**的結合。然而，RL訓練長期缺乏像預訓練那樣的「擴展定律」來預測：投入多少計算資源，能換來多少性能提升？

這篇筆記梳理2025-2026年間，學術界對RL訓練擴展定律的研究，以及從PPO到GRPO及其多個變種算法的技術演進脈絡。這些內容對理解為何2025-2026年的推理模型能力有如此顯著的突破至關重要。

---

## 從PPO到GRPO：為何需要新算法？

### 傳統RLHF（PPO）的問題

OpenAI在2022-2023年主導的RLHF（Reinforcement Learning from Human Feedback）使用**PPO（Proximal Policy Optimization）**算法。PPO需要三個模型同時訓練：

• **策略模型（Policy Model）**：實際生成回應的模型
• **獎勵模型（Reward Model）**：學習人類偏好的人工訓練模型
• **批評模型（Critic/Value Model）**：估算未來累計獎勋的價值模型

「三個模型」的訓練成本極高，而且獎勵模型的訓練本身就需要大量人類偏好標注數據。

### GRPO：消滅批評模型

**Group Relative Policy Optimization（GRPO）** 由DeepSeek團隊提出，本質上是PPO的簡化版：

| 組件 | PPO | GRPO |
|------|-----|------|
| 獎勵模型 | ✅ 需要 | ❌ 可不需要（視任務而定）|
| 批評模型 | ✅ 需要 | ❌ 用「組內相對基準」取代 |
| 人類偏好數據 | ✅ 需要 | ❌ 可不需要 |

**GRPO的核心原理**：
1. 對每個prompt，採樣多個回應（「組」）
2. 計算該組內每個回應的獎勵
3. 透過「相對於組平均值」的優勢函數（advantage）來更新策略
4. 不需要單獨訓練的批評模型

**優勢函數公式**：
```
A_i = (r_i - μ_group) / σ_group
```

GRPO的關鍵要求：**需要相對大的batch size**——每個prompt需要採樣足夠多的回應，以獲得可靠的優勢估計。

---

## GRPO的三大變種

### 1. GSPO（Group Sequence Policy Optimization）

**問題**：Token級別的政策比率（π_new/π_old）在訓練過程中具有高方差，導致不穩定。

**解決方案**：在**序列級別**用幾何平均計算重要性比率：
```
π(s)/π_ref(s) = exp(Σ log π(t) - Σ log π_ref(t)) / |s|
```

**效果**：訓練穩定性提升，樣本效率改善，整體性能提升。對大型MoE模型（如Qwen3-235B-A22B）特別有效。

---

### 2. DAPO（Dynamic Sampling Policy Optimization）

DAPO由位元組跳動團隊提出，是對vanilla GRPO的全面改進，解決了三個核心問題：

**問題一：熵崩潰（Entropy Collapse）**
• 現象：模型在RL訓練中變得過度確定性，缺乏探索多樣性
• 解決：「Clip Higher」——將裁剪邊界從[1-ε, 1+ε]改為非對稱的[1-ε_low, 1+ε_high]，ε_low=0.2，ε_high=0.28

**問題二：獎勵噪聲（Reward Noise）**
• 現象：訓練獎勋不斷波動，而非穩步上升
• 解決：動態採樣——對已達完美準確的prompt進行過濾，繼續採樣直到batch填滿

**問題三：訓練不穩定**
• 解決：
  - Token級別損失聚合（而非先做序列級別平均）
  - 基於長度的懲罰（軟性懲罰過長回應，而非硬截斷）

---

### 3. Dr. GRPO（Diagnosis of GRPO）

Dr. GRPO由研究團隊提出，識別了GRPO中兩個系統性偏差來源：

**偏差一：回應級別長度偏差**
• GRPO對回應長度進行標準化，引入基於長度的梯度偏差
• 短的正確答案可能獲得過高優勢

**偏差二：問題級別難度偏差**
• 標準差項（σ_group）對極易或極難的問題產生極端優勢值
• 容易的數學題（幾乎全部答對）→ σ_group趨近於0 → 優勢函數爆炸
• 解決：對σ_group設置最小值下限

---

## 可驗證獎勋強化學習（RLVR）

RLVR（Reinforcement Learning with Verifiable Rewards）是DeepSeek R1和o系列推理模型的核心訓練範式。

**核心思想**：放棄人類偏好數據，只用**符號可驗證的任務**來訓練：

• **數學**：答案正確與否由符號系統（如Python sympy、數學公式引擎）驗證
• **編程**：代碼是否通過單元測試由程序自動驗證

RLVR的關鍵洞察：**當獎勵可以被程式化驗證時，整個獎勵模型都可以被省略**——這讓訓練成本大幅下降，也避免了獎勵黑客（reward hacking）問題。

---

## RL訓練的擴展定律現況

與預訓練的Chinchilla定律（模型大小 + 訓練tokens的聯合擴展）不同，**RL訓練的擴展定律尚未完全確立**。研究現況：

• 預訓練：擴展定律相對成熟（模型參數數量、訓練tokens數量、算力預算的關係）
• RL訓練：每個任務的「好壞」定義不同，難以建立通用法則
• 推理模型：o系列和R1的成功暗示**推理時計算（inference-time compute）和RL訓練計算同樣重要**

Sebastian Raschka在2026年採訪中指出：推理模型的關鍵洞察是「兩種範式」：
1. **訓練擴展**：昂貴（500萬美元+），長期投資
2. **推理擴展**：在部署時投入額外計算（如運行3個回應取多數決）

---

## 實際應用啟示

### 訓練推理模型時：
• 優先選擇有明確「對錯」的任務（數學、編程）來使用RLVR
• 對開放式任務（寫作、創意），仍需要人類偏好數據或獎勵模型
• GRPO相比PPO訓練成本顯著更低，但需要更大的batch size來保證穩定性

### 選擇算法時：
• 簡單任務（回答封閉式問題）→ Vanilla GRPO足夠
• 大型MoE模型 → 考慮GSPO（序列級別優勢估計更穩定）
• 訓練過程出現熵崩潰 → DAPO的Clip Higher + 動態採樣
• 訓練獎勋波動大 → DAPO的動態採樣機制
• 對長度偏差敏感 → Dr. GRPO的長度歸一化修復

---

## 標籤
#AI #強化學習 #GRPO #LLM推理 #DAPO #GSPO #RLVR
