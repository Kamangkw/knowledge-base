---
title: LLM Fine-tuning：LoRA、RLHF、模型適配
date: 2026-04-20
tags: [Fine-tuning, LoRA, RLHF, PEFT, LLM]
description: 介紹大型語言模型微調的主要方法，包括 LoRA、QLoRA、RLHF 等技術的原理、優勢與應用場景。
---

# LLM Fine-tuning：LoRA、RLHF、模型適配

## 概述

Fine-tuning（微調）是指在預訓練模型基礎上，使用特定領域或任務的標註資料進一步訓練，使模型適應目標需求的技術。隨著 GPT、LLaMA、Claude 等大型語言模型的崛起，從頭訓練的成本極高，微調已成為模型客製化的主流方法。根據更新參數範圍的不同，微調可分為全參數微調（Full Fine-tuning）與參數高效微調（Parameter-Efficient Fine-tuning, PEFT）兩大類。

## 全參數微調

全參數微調更新模型所有權重，在足夠資料量下可達到最佳任務表現，但面臨以下挑戰：

- **計算成本高**：數十億參數的模型需要大量 GPU 資源。
- **災難性遺忘（Catastrophic Forgetting）**：新任務知識可能覆蓋預訓練知識。
- **儲存與部署成本**：每個任務需儲存完整模型權重。

因此，參數高效微調技術近年來受到廣泛關注。

## LoRA：低秩適配

LoRA（Low-Rank Adaptation）由 Hu et al. (2021) 在論文《LoRA: Low-Rank Adaptation of Large Language Models》中提出，核心思想是假設模型權重更新具有低秩結構。

具體而言，預訓練權重矩陣 $W \in \mathbb{R}^{d \times k}$ 的更新表示為：

```
ΔW = BA, 其中 B ∈ R^{d×r}, A ∈ R^{r×k}, r << min(d, k)
```

訓練時僅更新 A、B 矩陣，原始 W 保持凍結。推理時可將 ΔW 合併回 W，實現零額外推論延遲。

### QLoRA

QLoRA（Quantized LoRA）由 Dettmers et al. (2023) 提出，結合 4-bit 量化與 LoRA，大幅降低記憶體需求，使 65B 參數模型可在單張 48GB GPU 上微調。

## RLHF：人類反饋強化學習

RLHF（Reinforcement Learning from Human Feedback）是由 OpenAI 在 InstructGPT 論文中提出的微調範式，旨在讓模型生成符合人類偏好的回答。

### RLHF 三步驟

1. **監督微調（SFT）**：使用人類标注的高品質問答對，進行常規微調。
2. **獎勵模型訓練（Reward Model）**：訓練一個獎勵模型學習人類偏好。
3. **強化學習優化（PPO）**：以獎勵信號優化策略模型，常用 PPO（Proximal Policy Optimization）演算法。

RLHF 能顯著提升模型的有用性與安全性，是 ChatGPT、Claude 等對話模型的關鍵技術。

### 替代方法：DPO

直接偏好優化（Direct Preference Optimization, DPO）由 Rafailov et al. (2023) 提出，繞過獎勵模型，直接使用偏好資料優化策略，簡化了 RLHF 流程且效果相當。

## 其他 PEFT 方法

- **Adapter**：在 Transformer 層間插入小型適配器模組。
- **Prefix Tuning**：在輸入前新增可學習的前綴向量。
- **Prompt Tuning**：僅學習虛擬 token embeddings，參數量極小。

## 應用場景與選擇建議

| 方法 | 適用場景 | 資源需求 |
|------|----------|----------|
| LoRA | 一般領域微調、任務特化 | 中等 |
| QLoRA | 資源受限環境、大模型微調 | 低 |
| RLHF/DPO | 對話系統、偏好對齊 | 高 |
| Full Fine-tuning | 高資料量、追求最高精度 | 極高 |

## 小結

微調是將通用 LLM 適配至特定任務的核心技術。LoRA 系列方法以低成本實現高效特化，RLHF/DPO 則為對話對齊與安全性提供了可靠方案。選擇合適的微調策略，需在任務效果、計算資源與實施複雜度之間取得平衡。

---

**參考文獻**

- Hu, E.J., et al. "LoRA: Low-Rank Adaptation of Large Language Models." *arXiv:2106.09685*, 2021.
- Dettmers, T., et al. "QLoRA: Efficient Quantized Finetuning." *arXiv:2305.13014*, 2023.
- Ouyang, L., et al. "Training language models to follow instructions with human feedback." *arXiv:2203.02155*, 2022.
- Rafailov, R., et al. "Direct Preference Optimization: Your Language Model is Secretly a Reward Model." *arXiv:2305.18290*, 2023.
