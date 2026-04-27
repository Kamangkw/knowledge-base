---
title: Reflexion: 語言 Agent 的言語強化學習
date: 2026-04-22
source: 自行研究整理（主要參考：Shinn N et al., "Reflexion: Language Agents with Verbal Reinforcement Learning" arXiv:2303.11366）
tags: [LLM, Reflexion, Agent, 強化學習, 口語反饋]
description: 介紹Reflexion框架，讓語言Agent透過言語強化學習從失敗中自我改進，建立反思與修正的循環。
---

# Reflexion: 語言 Agent 的言語強化學習

## 基本資訊

| 項目 | 內容 |
|------|------|
| **標題** | Reflexion: Language Agents with Verbal Reinforcement Learning |
| **作者** | Noah Shinn, Federico Cassano, Edward Berman, Ashwin Gopinath, Karthik Narasimhan |
| **年份** | 2023 |
| **arXiv ID** | 2303.11366 |
| **發布機構** | Northeastern University / MIT |
| **原文連結** | https://arxiv.org/abs/2303.11366 |

---

## 研究背景與動機

大型語言模型（LLM）越來越多地被用作與外部環境互動的目標導向 Agent（如遊戲、編譯器、API）。然而，傳統的强化學習方法存在重大局限：

**傳統 RL 的問題**：
- 需要大量訓練樣本
- 需要昂貴的模型 fine-tuning
- 每次錯誤學習都需要更新權重
- 樣本效率極低

**核心問題**：如何讓語言 Agent 從試錯中快速、有效地學習，而不需更新模型參數？

**解決方案**：讓 Agent 用**語言而非權重**進行反思和學習。

---

## 核心方法 (Methodology)

### Reflexion 的核心思想

Reflexion 是一種**言語強化學習框架**——它不通過權重更新來強化語言 Agent，而是通過**語言反饋**來實現。

### 核心流程

```
┌─────────────────────────────────────────────────────────┐
│              Reflexion 框架                              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Execute（執行）                                      │
│     - Agent 嘗試完成任務                                 │
│     - 環境給出反饋信號                                   │
│                                                         │
│  2. Reflect（反思）                                       │
│     - Agent 用語言總結失敗原因                           │
│     - 分析反饋信號中的關鍵信息                           │
│                                                         │
│  3. Self-Correct（自我修正）                              │
│     - 根據反思更新內部記憶                               │
│     - 修改策略以避免重複錯誤                             │
│                                                         │
│  4. Iterate（迭代）                                      │
│     - 帶着新的反思記憶重新嘗試                           │
│     - 直到任務完成或達到最大步數                         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 三種反饋類型

| 反饋類型 | 描述 | 示例 |
|---------|------|------|
| **Scalar Feedback** | 簡單的數值評估 | ✓/✗, 1-10 分 |
| **Self-Evaluation** | Agent 自己評估 | 「我的代碼有語法錯誤」 |
| **External Feedback** | 環境/工具反饋 | 編譯器錯誤信息 |

### 記憶機制

Reflexion 使用**情景記憶緩衝區（Episodic Memory Buffer）**：
- 保存 Agent 的反思和決策歷史
- 在後續嘗試中檢索相關記憶
- 指導更好的決策

---

## 核心貢獻 (Main Findings)

### 1. HumanEval 編碼基準測試

| 方法 | Pass@1 準確率 |
|------|-------------|
| GPT-4（無反射） | 80.3% |
| **Reflexion + GPT-4** | **91.0%** |
| GPT-4 + 多次嘗試 | 86.5% |

> Reflexion 將 GPT-4 在編碼任務上的表現從 80% 提升至 91%，超越之前所有方法。

### 2. 跨任務泛化能力

Reflexion 在多個領域展現顯著提升：

| 任務類型 | 任務名稱 | 提升幅度 |
|---------|---------|---------|
| 順序決策 | AlfWorld | +22% |
| 編碼 | HumanEval | +10.7% |
| 語言推理 | Big-Bench Hard | +6.9% |

### 3. 零樣本反思能力

- Agent 能夠從單次失敗中提取通用教訓
- 將「犯錯經驗」轉化為「未來策略」
- 不需要額外的訓練數據

### 4. 靈活性

- **兼容各反饋類型**：Scalar、自由語言、內部/外部
- **兼容各 Agent 類型**：單輪、多輪、工具使用型
- **無需參數更新**：所有學習都在推理時發生

---

## 為什麼重要 (Why It Matters)

1. **突破傳統 RL 的瓶頸**：
   - 樣本效率從極低提升到極高
   - 不需要昂貴的 fine-tuning

2. **語言作為認知工具**：
   - 首次展示語言本身可以作為學習機制
   - 為「語言作為推理引擎」提供新證據

3. **實用性強**：
   - 可應用於任何現有 LLM
   - 只需在 prompt 中添加反思步驟

4. **為 Agent 系統奠定基礎**：
   - 讓 Agent 能從錯誤中學習
   - 這是構建真正自主 Agent 的關鍵能力

---

## 實際應用場景

1. **軟件開發助手**
   - 自動修復代碼錯誤
   - 從編譯失敗中學習修正策略

2. **研究助手**
   - 自動設計和調整實驗
   - 從結果中反思並改進方法

3. **多步驟自動化**
   - 需要多輪試錯的複雜任務
   - 如：數據分析、報告生成

4. **決策支持系統**
   - 從錯誤決策中學習
   - 逐步優化決策質量

---

## 程式碼概念

```python
# Reflexion 概念實現
def reflexion_agent(task, max_trials=3):
    memory = []  # 反思記憶
    feedback_history = []
    
    for trial in range(max_trials):
        # 根據記憶構建 prompt
        prompt = build_prompt(task, memory)
        
        # 執行任務
        result = execute(prompt)
        
        # 獲取反饋
        feedback = get_feedback(result)
        
        # 反思
        if not is_success(feedback):
            reflection = reflect(result, feedback)
            memory.append(reflection)
            feedback_history.append(feedback)
        else:
            return result
    
    return failure
```

---

## 局限性 (Limitations)

1. **語言反饋質量依賴**：如果環境不提供清晰反饋，反思效果受限
2. **記憶衰減**：隨着任務複雜度增加，記憶檢索可能不準確
3. **最大嘗試次數**：需要在計算成本和成功率之間權衡
4. **並非通用解決方案**：某些任務（如純數學證明）難以提供可操作的反饋

---

## 相關研究

- **Toolformer**：為 Agent 提供工具使用能力
- **ReAct**：結合推理和行動的框架
- **Tree of Thoughts**：規劃和探索的框架
- **Self-Consistency**：多路徑推理和投票

---

## 摘要

Reflexion 是一種創新的言語强化學習框架，它允許語言 Agent 通過語言反思而非權重更新來從錯誤中學習。這種方法既簡單又高效——只需添加「反思步驟」就能顯著提升 Agent 在編碼、決策、推理等多種任務上的表現。Reflexion 為構建更智能、更自主的 AI Agent 系統提供了重要思路，並展示了語言本身作為認知學習工具的巨大潛力。

---

*筆記整理日期：2026年4月22日*
