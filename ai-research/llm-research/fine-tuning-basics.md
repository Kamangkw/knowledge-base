---
title: LLM Fine-tuning：LoRA、RLHF與模型適配
date: 2026-04-20
source: arXiv:2106.09685 (Hu et al., LoRA); arXiv:2203.02155 (Ouyang et al., InstructGPT)
tags: [Fine-tuning, LoRA, RLHF, DPO, 模型訓練, LLM, AI]
---

# LLM Fine-tuning：LoRA、RLHF與模型適配

## 概述

Fine-tuning（微調）是將預訓練大語言模型（LLM）適配到特定任務或領域的關鍵技術。儘管 prompt engineering 能在一定程度上引導 LLM 的行為，但對於需要穩定、深入地改變模型輸出風格或能力結構的應用場景，fine-tuning 往往是不可或缺的。

傳統的 full fine-tuning 需要對模型的所有參數進行更新，這在計算資源和存儲成本上都極為昂貴。以 GPT-3 為例，1750 億參數的模型完全更新一次需要數百萬美元的計算成本。引數效率微調（Parameter-Efficient Fine-Tuning, PEFT）技術的出現，讓中小型團隊也能負擔得起 LLM 的定制化訓練。

## 全量微調 vs 輕量化微調

### 全量微調（Full Fine-tuning）

全量微調是指對預訓練模型的所有參數進行更新。這種方法的優點是能充分挖掘預訓練模型的能力上限，缺點顯而易見：

- **計算成本極高**：需要多 GPU 集群，單次訓練可能耗費數十萬至數百萬美元
- **存儲成本高**：每次訓練都需要保存一份完整的模型權重
- **災難性遺忘**：模型可能遺忘預訓練階段學到的通用能力和知識
- **過擬合風險**：在小數據集上全量微調容易過擬合

### PEFT 方法的興起

為了克服全量微調的種種問題，學術界和工業界發展出了多種輕量化微調技術：

**LoRA (Low-Rank Adaptation)**：通過在原模型權重旁添加低秩矩陣來近似權重更新，大幅減少可訓練參數數量。

**Adapter**：在 Transformer 層之間插入小型適配器模塊，只訓練這些新增模組。

**Prefix Tuning**：在輸入前添加可學習的虛擬 token 前綴，凍結原模型參數。

**Prompt Tuning**：僅學習一組軟提示（soft prompts），是一種極端的參數效率方法。

## LoRA：低秩適配的核心原理

### 核心思想

LoRA（Hu et al., 2021）的核心思想源於一個關鍵假設：神經網絡在 fine-tuning 過程中的權重更新矩陣往往是低秩的。也就是說，這個更新矩陣的有效自由度遠小於其實際參數數量。

基於這一假設，LoRA 不直接學習原權重矩陣 W 的更新 ΔW，而是學習 W 的低秩分解表示。具體來說：

對於預訓練權重矩陣 W₀ ∈ ℝ^(d×k)，LoRA 將其更新表示為：
W = W₀ + ΔW = W₀ + BA

其中：
- B ∈ ℝ^(d×r) 和 A ∈ ℝ^(r×k) 是新添加的可訓練低秩矩陣
- r << min(d, k) 是低秩維度（通常設為 8、16、32 或 64）
- 訓練時 W₀ 保持凍結，只訓練 A 和 B

在推理時，可以將 W₀ + BA 合併為一個矩陣，完全沒有額外的推理延遲。

### LoRA 的實際配置

LoRA 的配置通常包括以下關鍵參數：

**rank (r)**：低秩維度，決定了可訓練參數的数量。較高的 r 能捕捉更豐富的任務特定信息，但也需要更多計算資源。實際應用中，r=8~64 是常見範圍。

**alpha**：縮放因子，通常設為 r 的倍數（如 2r）。這個超參數影響著訓練初期學習率的設置。

**dropout**：LoRA 分支的 dropout 率，通常設為 0.05~0.1。

**target_modules**：指定哪些權重矩陣需要添加 LoRA 適配。對於 LLM，通常是 attention 中的 Query (q_proj) 和 Value (v_proj) 權重。

### QLoRA：更進一步的量化

QLoRA（Dettoni et al., 2023）是 LoRA 的重要進化版本，它將量化技術與 LoRA 相結合，進一步降低了訓練的記憶體需求。

QLoRA 的核心技術包括：
- **NF4 (4-bit NormalFloat)**：一種專為神經網絡權重設計的 4 位量化格式
- **雙重量化**：對量化常數本身也進行量化
- **分頁優化器**：處理梯度更新過程中的記憶體峰值

QLoRA 的創新在於：即使將模型量化至 4-bit 精度，通過 LoRA 適配器的補償，依然能保持接近全量精度的模型性能。這使得在單個消費級 GPU（如 48GB 的 RTX 6000）上 fine-tuning 65B 參數的模型成為可能。

## RLHF：從人類反饋中學習

### RLHF 的背景

即使是最先進的 LLM，也往往難以產生完全符合人類期望的輸出。預訓練目標（預測下一個 token）與人類期望之間存在根本性的目標差異。RLHF（Reinforcement Learning from Human Feedback）提供了一種讓模型學習人類偏好的方法。

InstructGPT（Ouyang et al., 2022）的工作表明，RLHF 能顯著提升模型的有用性（helpful）、無害性（harmless）和誠實性（honest）。這一發現直接催生了 ChatGPT 的成功。

### RLHF 的三階段流程

**階段一：監督微調（Supervised Fine-Tuning, SFT）**
首先，在一個高質量的示範數據集上對預訓練模型進行微調。這個數據集通常由人工編寫，展示了理想輸出應有的格式和風格。

**階段二：訓練獎勵模型（Reward Model, RM）**
收集人類對同一 prompt 不同輸出的人工排序數據，訓練一個獎勵模型來預測人類偏好。這個獎勵模型通常是對 LLM 進行微調，添加一個回歸頭來輸出單一標量獎勵。

**階段三：強化學習微調**
使用 Policy Gradient 方法（如 PPO 算法）優化 LLM，使其生成能夠最大化獎勵模型打分的輸出。同時添加 KL 散度約束，防止模型偏離 SFT 模型太遠。

### RLHF 的挑戰

RLHF 面臨著多個挑戰：

**人工標註成本**：高質量的人類反饋數據需要大量時間和金錢來收集。

**獎勵黑客（Reward Hacking）**：模型可能發現獎勵函數的漏洞，產生在獎勵模型看來高分但在人類看來質量低下的輸出。

**多樣性崩潰（Mode Collapse）**：強化學習階段可能導致模型喪失輸出多样性，只產生"平均"的最優輸出。

## DPO：直接偏好優化

### DPO 的原理

直接偏好優化（Direct Preference Optimization, Rafailov et al., 2023）提出了一種優雅的方法來簡化 RLHF。DPO 的核心洞見在於：可以直接使用偏好數據，通過一個精心設計的損失函數來優化模型，繞過獎勵模型的訓練和強化學習的複雜性。

DPO 的損失函數直觀上鼓勵模型：
- 對人類偏好的輸出（chosen）給予較高的概率
- 對人類不偏好的輸出（rejected）給予較低的的概率

DPO 的優勢在於：
- 訓練更加穩定，收斂更快
- 不需要單獨的獎勵模型
- 減少了超參數的數量

### 實驗結果

在多個偏好數據集上的實驗表明，DPO 能達到與 PPO-based RLHF 相當甚至更好的效果，同時訓練過程更加簡單高效。這使得 DPO 迅速成為 RLHF 的流行替代方案。

## 何時使用 Fine-tuning

### Fine-tuning 的適用場景

- 需要定制的輸出風格或語氣
- 需要模型掌握特定領域的專業知識
- 需要稳定、一致地執行特定任務
- 需要在特定語言或方言上優化性能
- 需要压低延遲或推理成本

### Prompt Engineering 的適用場景

- 快速原型驗證
- 任務簡單且樣式相對標準
- 沒有足够的訓練數據
- 需要頻繁更換任務

### 實際建議

1. **從 Prompt Engineering 開始**：在投入 fine-tuning 之前，先用精心設計的 prompt 充分測試模型的能力上限。

2. **評估數據需求**：LoRA 通常需要數百到數千個高質量示例；RLHF 需要數萬到數十萬的人類偏好比較。

3. **評估計算資源**：QLoRA 能在單卡上訓練 7B-70B 模型；全量 RLHF 需要多卡集群。

4. **持續評估**：建立可靠的評估指標和測試集，追蹤 fine-tuning 過程中的性能變化。

## 主流 Fine-tuning 框架

**Hugging Face PEFT**：最流行的 PEFT 工具庫，支援 LoRA、Prefix Tuning、Prompt Tuning、Adapter 等多種方法。

**Axolotl**：專為 LLM fine-tuning 設計的訓練框架，支援多種預訓練模型和微調方法。

**LLaMA-Factory**：針對 LLaMA 系列模型的高效微調工具，提供 WebUI 界面。

**DeepSpeed-Chat**：Microsoft 開源的 RLHF 訓練框架，支援完整的 InstructGPT 流程。

## 結論

Fine-tuning 是將通用 LLM 適配至特定應用的核心技術。LoRA/QLoRA 大幅降低了微調的計算和存儲成本，使定制化 LLM 變得更加普及。RLHF 和 DPO 等偏好學習方法，則為讓模型更好地符合人類期望提供了有效途徑。

選擇何種 fine-tuning 方法，需要根據具體任務、數據規模和計算資源來綜合考慮。合理的策略往往是：先用 Prompt Engineering 驗證可行性，再用 LoRA/QLoRA 進行輕量化微調，最後根據需要考慮 RLHF/DPO 來進一步優化人類偏好。

## 延伸閱讀

- Hu et al., "LoRA: Low-Rank Adaptation of Large Language Models", arXiv:2106.09685
- Dettoni et al., "QLoRA: Efficient Finetuning of Quantized LLMs", arXiv:2305.13014
- Ouyang et al., "Training language models to follow instructions with human feedback", arXiv:2203.02155
- Rafailov et al., "Direct Preference Optimization: Your Language Model is Secretly a Reward Model", arXiv:2305.18290

---

*本 notes 由 Hermes Agent 編寫，僅供學術參考。*
