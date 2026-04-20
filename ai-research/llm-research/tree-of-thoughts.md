# Tree of Thoughts: Deliberate Problem Solving with Large Language Models

## 基本資訊

| 項目 | 內容 |
|------|------|
| **標題** | Tree of Thoughts: Deliberate Problem Solving with Large Language Models |
| **作者** | Yao, Shunyu; Yu, Dian; Zhao, Jeffrey; Shafran, Izhak; Griffiths, Thomas L. |
| **年份** | 2023 |
| **arXiv ID** | 2305.10601 |
| **發布機構** | Princeton University, DeepMind |
| **原文連結** | https://arxiv.org/abs/2305.10601 |

---

## 研究背景與動機

語言模型在各種任務中的部署越來越廣泛，但在推理過程中仍然受限於 token-level、left-to-right 的決策方式。這種限制意味著模型在需要探索、策略性預測或初始決策扮演關鍵角色的任務中會遇到困難。

傳統的 Chain-of-Thought (CoT) prompting 雖然能引導模型展現推理過程，但仍是線性思維模式，無法有效進行探索和規劃。

---

## 核心貢獻 (Main Findings)

### 1. Tree of Thoughts (ToT) 框架
ToT 框架將 CoT 的線性思維擴展為樹狀結構，讓語言模型能夠：
- 在每個思維步驟生成多個候選方向
- 評估每個分支的進展與前景
- 進行前瞻性規劃（lookahead）以避免不良決策
- 在必要時回溯或改變策略

### 2. 系統架構四階段
ToT 框架包含四個關鍵階段：

```
1. Thought Decomposition (思維分解)
   - 將問題分解為有意義的中間步驟

2. Thought Generator (思維生成)
   - 樣本模式：生成多個候選思維
   - 貪婪模式：生成單一最佳思維

3. State Evaluator (狀態評估)
   - 讓 LM 自己評估每個狀態的進展
   - 整合外部工具進行驗證

4. Search Algorithm (搜索演算法)
   - BFS (廣度優先)
   - DFS (深度優先)
   - Beam Search
```

### 3. 實驗結果
在三個任務上測試 ToT：
- **24點遊戲** (Creative Writing & 24 Game)：從 6% 提升至 74%
- **迷你縱橫字謎** (Mini Crosswords)：從 16% 提升至 60%
- **格式化輸出**：顯著改善複雜輸出結構

### 4. 與現有方法的對比
| 方法 | 能力 |
|------|------|
| Input-Output | 無推理能力 |
| Chain-of-Thought | 線性推理，無探索 |
| Self-Consistency + CoT | 結果一致性，但仍是線性 |
| Tree of Thoughts | 多路徑探索、策略規劃、回溯能力 |

### 5. 解決問題類型
ToT 特別適合以下類型問題：
- 需要「試錯」過程的任務
- 初始決策會影響最終結果的問題
- 需要戰略性思考的複雜推理
- 搜索空間非線性的挑戰

---

## 為什麼重要 (Why It Matters)

1. **突破線性思維限制**：首次將語言模型的推理從線性扩展為樹狀結構
2. **賦予規劃能力**：模型能夠進行前瞻性和回溯性決策
3. **通用框架**：適用於各種任務和語言模型
4. **理論基礎**：為 LLM 推理提供新的理論框架

---

## 實際應用場景

1. **複雜問題解決**
   - 數學證明
   - 邏輯推理
   - 程式碼生成與除錯

2. **創意寫作**
   - 需要迭代改進的長文本生成
   - 多約束條件下的內容創作

3. **決策系統**
   - 需要評估多種方案的業務決策
   - 策略遊戲 AI

4. **代理系統 (Agent Systems)**
   - 為 AI Agent 提供規劃能力
   - Tool-augmented LLM 的核心架構

---

## 程式碼與資源

- GitHub: `https://github.com/princeton-nlp/tree-of-thought-llm`
- 支援多種語言模型的實現

---

## 相關研究

- Chain-of-Thought Prompting (Wei et al., 2022)
- Self-Consistency (Wang et al., 2022)
- Reasoning with Language Model (PAL, ReAct, etc.)

---

## 摘要

Tree of Thoughts 框架是 LLM 推理領域的重要突破。它通過將線性思維擴展為樹狀結構，賦予語言模型探索、規劃和回溯的能力。這種框架對於構建更強大的 AI 代理系統和解決複雜推理任務具有重要意義。

---

*筆記整理日期：2026年4月17日*
