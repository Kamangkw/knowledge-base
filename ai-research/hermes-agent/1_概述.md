---
title: Hermes Agent：开源 AI Agent 框架
date: 2026-05-02
source: 自行研究整理（主要參考：https://hermes-agent.nousresearch.com/docs/、SKILL.md）
tags: [Hermes-Agent, AI-Agent, Nous-Research, 开源框架, 工具调用]
description: 深入介紹 Hermes Agent——Nous Research 開發的開源 AI Agent 框架，涵蓋其核心設計哲學、Skills 系統、Memory 機制、Multi-Agent 協作，以及相較於 Claude Code、OpenClaw 等工具的獨特優勢。
---

# Hermes Agent：开源 AI Agent 框架

## 概述

Hermes Agent 是由 Nous Research 開發的開源 AI Agent 框架，與 Anthropic 的 Claude Code、OpenAI 的 Codex 屬於同類產品——都是能夠透過工具調用（Tool Calling）與系統互動的自主編碼與任務執行 Agent。與封閉式商業方案不同，Hermes Agent 完全開源，支持自架部署，且支持 20+ 種 LLM Provider，讓用戶不受單一模型限制。

**核心差異化特點：**

- **Self-improving Skills 系統** — 每完成複雜任務，可將流程存為可重用 Skill，隨使用時間累積，Agent 會愈來愈熟悉你的項目與偏好
- **跨平台 Messaging Gateway** — 同一個 Agent 可同時運行在 Telegram、Discord、Slack、WhatsApp、Email 等 10+ 平台，有完整工具訪問權限，不只是聊天機器人
- **Provider-agnostic** — 可隨時切換不同模型與 Provider（OpenRouter、Anthropic、DeepSeek、Local Models 等），無需改變其他配置
- **Profiles 隔離** — 可運行多個獨立 Hermes 實例，各自擁有完全隔離的 Config、Skills、Memory 與 Session

從實用角度來看，Hermes Agent 適合需要長期運行 AI 助理的用戶——它不只是回答問題，而是能真正自動化重複工作、記憶跨會話偏好、並通過 Skills 系統持續學習更好的工作方式。

## 核心設計哲學

### Skills：持久化學習的基礎

Hermes Agent 最獨特之處在於其 Skills 系統。傳統 Agent 每個會話都是全新的，無法從過去的經驗中學習。而 Hermes 允許你在完成複雜任務後，將整個工作流程保存為一個 Skill 文件（SKILL.md），下次遇到類似任務時可直接加載使用。

**Skill 的價值：**

- **累積經驗**：每個 Skill 都是過去解決問題的結晶
- **快速啟動**：新會話無需重新解釋背景，直接加載 Skill 即可
- **跨會話記憶**：不同於短期對話歷史，Skills 持久存在
- **可分享**：Skills 可發布到 Skills Hub 與其他用戶共享

**觸發條件寫作原則：** Skill 的描述（Description）應以「Use when ...」開頭，精確描述觸發情境而非具體任務。

### Memory：跨會話的持久上下文

Memory 系統讓 Agent 能記住用戶是誰、偏好是什麼、環境配置等長期信息。支持多種 Memory Backend：內置（默認）、Honcho、Mem0 等。Memory 內容會被注入每個新會話的系統提示中。

### 雙階段壓縮機制

當對話接近 token 限制時，Hermes 會自動觸發上下文壓縮（Context Compression）：

1. **Threshold（50%）**：對話長度達到 context window 50% 時觸發
2. **Target Ratio（20%）**：壓縮後目標為原長度的 20%
3. **手動觸發**：隨時可用 `/compress` 命令手動壓縮

## 工具生態

### 工具集（Toolsets）

| Toolset | 功能 |
|---------|------|
| `web` | 網頁搜索與內容提取 |
| `browser` | 瀏覽器自動化 |
| `terminal` | Shell 命令與進程管理 |
| `file` | 文件讀寫、搜索、补丁 |
| `code_execution` | 沙盒 Python 執行 |
| `vision` | 圖像分析 |
| `delegation` | Subagent 任務委派 |
| `cronjob` | 定時任務管理 |
| `messaging` | 跨平台消息發送 |
| `homeassistant` | 智能家居控制 |

### 自定義工具開發

只需 3 個文件即可添加新工具：

1. **工具文件**：`tools/your_tool.py` — 實現工具邏輯並調用 `registry.register()`
2. **工具集配置**：添加到 `toolsets.py` 的 `_HERMES_CORE_TOOLS` 列表
3. **自動發現**：任何含 `registry.register()` 調用的 `tools/*.py` 文件會被自動導入

## Multi-Agent 協作

### delegate_task 與獨立進程

| | `delegate_task` | 獨立 Hermes 進程 |
|-|-----------------|-----------------|
| 隔離性 | 獨立對話，共享進程 | 完全獨立進程 |
| 持續時間 | 分鐘級 | 小時/天級 |
| 工具訪問 | 父進程工具子集 | 完整工具訪問 |
| 適用場景 | 快速並行子任務 | 長期自主任務 |

**delegate_task 最佳實踐：**

- 每個 subagent 應有完整、獨立的上下文
- 優先用於需要並行處理的獨立子任務
- 結果需二次確認

### Worktree 模式：並行開發

`hermes -w` 開啟隔離的 git worktree 模式，適合多個 Agent 同時開發同一個 Repo 而不產生 git conflicts。

## 自動化：Cron Jobs

Cron Jobs 讓 Hermes 能執行週期性任務，適合：

- 每日資訊收集（新聞、天氣、股票）
- 定期數據備份
- 自動化研究工作流
- 定時消息提醒

**工作流鏈接（Chain）：** 多個 Cron Job 可以串聯，一個的輸出可作為下一個的輸入。

## 與其他框架的比較

| 維度 | Hermes Agent | Claude Code | OpenClaw |
|------|-------------|-------------|----------|
| 開源 | ✅ 完全開源 | ❌ 封閉 | ✅ 開源 |
| Provider | 20+ 靈活切換 | 主要是 Anthropic | 多種 |
| Skills 系統 | ✅ 持久化學習 | ❌ | ✅ |
| 跨平台 Gateway | ✅ 10+ 平台 | ❌ | ❌ |

Hermes 的核心優勢在於**開源 + 跨平台 + Skills 持久化學習**的組合。

## 實用技巧

1. **善用 `/skill`**：遇到複雜任務前，先看有沒有相關 Skill 可加載
2. **主動創建 Skill**：每完成一次複雜工作流，主動問「要不要存成 Skill」
3. **Session 恢復**：`/resume` 或 `--continue` 可隨時恢復中斷的任務
4. **並行加速**：`delegate_task` 適合需要同時 research 多個方向的場景
5. **定期檢查 Cron Jobs**：確保自動化任務正常運行
6. **`--yolo` 慎用**：繞過所有確認，適合已知安全的批量操作

## 資源

- **官方文檔**：https://hermes-agent.nousresearch.com/docs/
- **GitHub**：https://github.com/NousResearch/hermes-agent

---

*最後更新：2026年5月2日*
