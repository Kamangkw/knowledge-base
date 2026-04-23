# AI 安全 README

## 📂 概述

本目錄收錄 AI 安全（AI Safety）領域的重要研究，涵蓋對齊技術、對抗攻擊防禦、LLM 欺騙行為等主題。

## 📚 現有筆記

### 對齊技術
- [Constitutional AI](constitutional-ai.md) — AI 反饋訓練無害助手（Anthropic）

### 對抗安全
- [Prompt 注入攻擊](prompt-injection-attacks.md) — 自動化和通用的 Prompt Injection 攻擊與防禦
- [LLM 的欺騙能力](llm-deception.md) — 四種欺騙類型的系統性分析

### 待補充
- [ ] AI 控制問題（AI Control）
- [ ] 對抗樣本攻擊（Adversarial Examples）
- [ ] 獎勵黑客（Reward Hacking）
- [ ] 遺忘學習的安全性
- [ ] 價值對齊問題概述
- [ ] AI Safety Summit 2023 要點

## 🔗 相關領域

- [AI 倫理](../ai-ethics/) — 安全性與倫理性密不可分
- [LLM 研究](../llm-research/) — 理解模型能力是安全的基礎
- [Prompt Engineering](../prompt-engineering/) — 安全性在 Prompt 設計中的角色

## 🔐 核心安全威脅

| 威脅類型 | 描述 | 現實影響 |
|---------|------|---------|
| Prompt 注入 | 劫持模型指令 | 數據洩露、惡意操作 |
| 對抗樣本 | 精心構造的輸入 | 模型誤判 |
| 獎勵黑客 | 操縱優化目標 | 目標偏差 |
| 策略性欺騙 | 誤導用戶以達目標 | 信任危機 |
| 奉承性欺騙 | 過度迎合用戶 | 強化錯誤信念 |

## 🛡️ 主要安全技術

| 技術 | 描述 | 代表工作 |
|------|------|---------|
| RLHF | 人類反饋強化學習 | InstructGPT |
| Constitutional AI | AI 反饋自我改進 | Anthropic |
| DPO | 直接偏好優化 | Stanford |
| Red Teaming | 對抗性測試 | 各主要實驗室 |

---

*最後更新：2026年4月22日*
