# AI 研究 README

## 📂 概述

本目錄涵蓋 AI 研究的核心領域：Prompt Engineering、LLM 研究、AI 倫理和 AI 安全。

## 📚 四大領域

### 🤖 Prompt Engineering
讓 LLM 更好用——研究如何設計有效的提示詞。

| 筆記 | 主題 |
|------|------|
| [Chain-of-Thought](prompt-engineering/chain-of-thought.md) | 思維鏈提示 — 推理能力的關鍵突破 |
| [Toolformer](prompt-engineering/toolformer.md) | 工具使用 — LLM 自學使用外部 API |

### 🧠 LLM 研究
深入理解大型語言模型的能力、局限性和訓練方法。

| 筆記 | 主題 |
|------|------|
| [LLM Agents](llm-research/llm-agents.md) | Agent 系統 — LLM 作為自主代理 |
| [RAG](llm-research/rag-retrieval.md) | 檢索增強生成 — 結合向量數據庫 |
| [Fine-tuning](llm-research/fine-tuning-basics.md) | 微調技術 — LoRA、RLHF 等 |
| [Self-Consistency](llm-research/self-consistency.md) | 自一致性 — 多路徑推理投票 |
| [Tree of Thoughts](llm-research/tree-of-thoughts.md) | 思維樹 — 探索式問題解決 |
| [DPO](llm-research/dpo.md) | 直接偏好優化 — 簡化對齊訓練 |
| [Reflexion](llm-research/reflexion.md) | 言語強化學習 — 從錯誤中反思 |

### ⚖️ AI 倫理
探討 AI 的道德、社會影響和治理框架。

| 筆記 | 主題 |
|------|------|
| [東西方 AI 倫理比較](ai-ethics/confucius-eu-ai-ethics.md) | 儒家 vs 康德 — 跨文化倫理分析 |
| [超越原則主義](ai-ethics/beyond-principlism.md) | 科研 AI 倫理 — 實用主義框架 |

### 🛡️ AI 安全
確保 AI 系統可靠、可控、不被濫用。

| 筆記 | 主題 |
|------|------|
| [Constitutional AI](ai-safety/constitutional-ai.md) | 原則約束 — AI 反饋自我改進 |
| [Prompt 注入攻擊](ai-safety/prompt-injection-attacks.md) | 對抗攻擊 — 自動化 Prompt Injection |
| [LLM 欺騙能力](ai-safety/llm-deception.md) | 欺騙分析 — 四種欺騙類型系統 |

## 🔗 推理能力進化史

```
Chain-of-Thought (2022)
    ↓
Self-Consistency + Tree of Thoughts (2023)
    ↓
Toolformer + Reflexion (2023)
    ↓
RAG + Fine-tuning (2023)
    ↓
Constitutional AI + DPO (2022-2023)
    ↓
更安全、更智能的 LLM (2024+)
```

## 📈 關鍵研究方向

- **Prompt Engineering**：如何從模型中榨取最大價值
- **Reasoning**：從 CoT 到 AGI 的推理能力之路
- **Alignment**：確保 AI 行為符合人類意圖
- **Safety**：防範對抗攻擊和意外行為

---

*最後更新：2026年4月22日*
