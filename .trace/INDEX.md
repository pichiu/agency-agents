# The Agency — 專案總覽與速查

## 一句話總結

**The Agency** 是一個開源的 AI agent persona 函式庫，收錄 144+ 個跨 12+ 個專業分部的 AI agent 定義檔案（`.md` + YAML frontmatter），透過 Bash toolchain 將這些定義轉換並部署到 Claude Code、Cursor、Aider、Windsurf 等主流 AI coding 工具中，讓開發者可以在任何 AI coding session 中啟用具備深度領域知識、鮮明個性和可量化交付成果的 AI 專家助手。

---

## 技術棧總覽

| 類別 | 技術 | 版本/說明 | 用途 |
|------|------|-----------|------|
| 文件格式 | Markdown (.md) | CommonMark | Agent 定義主格式 |
| Metadata | YAML frontmatter | — | Agent 結構化 metadata |
| 腳本語言 | Bash | 3.2+（相容 macOS） | 轉換/安裝/驗證 toolchain |
| CI/CD | GitHub Actions | ubuntu-latest | PR 自動 lint 驗證 |
| 國際化 | PowerShell + JSON | — | Windows 環境 zh-CN 本地化 |
| 並行化 | xargs -P | GNU/BSD xargs | 多工具並行轉換/安裝 |

---

## 關鍵指令速查

```bash
# 轉換所有工具格式（先執行這步）
./scripts/convert.sh

# 安裝到偵測到的工具（互動模式）
./scripts/install.sh

# 安裝到指定工具
./scripts/install.sh --tool claude-code
./scripts/install.sh --tool cursor
./scripts/install.sh --tool aider

# 非互動安裝（CI/自動化）
./scripts/install.sh --no-interactive --tool all

# 並行加速
./scripts/convert.sh --parallel
./scripts/install.sh --no-interactive --parallel

# 只轉換特定工具
./scripts/convert.sh --tool gemini-cli

# 驗證 agent 格式
./scripts/lint-agents.sh
./scripts/lint-agents.sh path/to/agent.md

# 手動複製（最簡單的 Claude Code 安裝）
cp engineering/*.md ~/.claude/agents/
```

---

## 支援的 AI 工具平台

| 工具 | 安裝方式 | 目標路徑 |
|------|----------|----------|
| **Claude Code** | 直接複製（無需轉換） | `~/.claude/agents/` |
| **GitHub Copilot** | 直接複製（無需轉換） | `~/.github/agents/` |
| **Cursor** | `convert.sh` → `.mdc` | `.cursor/rules/` |
| **Aider** | `convert.sh` → 單一 `CONVENTIONS.md` | `./CONVENTIONS.md` |
| **Windsurf** | `convert.sh` → 單一 `.windsurfrules` | `./.windsurfrules` |
| **OpenCode** | `convert.sh` → `.md` + mode:subagent | `.opencode/agents/` |
| **Antigravity** | `convert.sh` → `SKILL.md` | `~/.gemini/antigravity/skills/` |
| **Gemini CLI** | `convert.sh` → extension + `SKILL.md` | `~/.gemini/extensions/agency-agents/` |
| **OpenClaw** | `convert.sh` → `SOUL.md` + `AGENTS.md` + `IDENTITY.md` | `~/.openclaw/agency-agents/` |
| **Qwen Code** | `convert.sh` → `.md` SubAgent | `.qwen/agents/` |
| **Kimi Code** | `convert.sh` → `agent.yaml` + `system.md` | `~/.config/kimi/agents/` |

---

## Agent 分部一覽

| 分部 | 目錄 | 代表 Agent | 數量 |
|------|------|------------|------|
| 工程 Engineering | `engineering/` | Frontend Developer, Backend Architect, Security Engineer | 23 |
| 行銷 Marketing | `marketing/` | Growth Hacker, SEO Specialist, Douyin Strategist | 25+ |
| 特殊 Specialized | `specialized/` | Agents Orchestrator, MCP Builder, Blockchain Security Auditor | 40+ |
| 測試 Testing | `testing/` | Reality Checker, Evidence Collector, Performance Benchmarker | 8 |
| 設計 Design | `design/` | UI Designer, UX Researcher, Brand Guardian | 8 |
| 遊戲開發 Game Dev | `game-development/` | Unity Architect, Godot Scripter, Blender Addon Engineer | 17+ |
| 銷售 Sales | `sales/` | Deal Strategist, Discovery Coach, Pipeline Analyst | 8 |
| 付費媒體 Paid Media | `paid-media/` | PPC Strategist, Paid Media Auditor, Ad Creative Strategist | 7 |
| 支援 Support | `support/` | Support Responder, Analytics Reporter, Infrastructure Maintainer | 6 |
| 空間運算 Spatial | `spatial-computing/` | XR Interface Architect, visionOS Spatial Engineer | 6 |
| 專案管理 PM | `project-management/` | Project Shepherd, Studio Producer, Jira Workflow Steward | 6 |
| 財務 Finance | `finance/` | Financial Analyst, Tax Strategist, Investment Researcher | 5 |
| 產品 Product | `product/` | Product Manager, Sprint Prioritizer, Trend Researcher | 5 |
| 學術 Academic | `academic/` | Anthropologist, Historian, Narratologist | 5 |
| 策略 Strategy | `strategy/` | NEXUS 框架文件、playbooks、runbooks | — |

---

## 文件地圖

| 文件 | 說明 |
|------|------|
| [`INDEX.md`](INDEX.md)（本文件）| 專案總覽、指令速查、術語表 |
| [`CODEBASE_MAP.md`](CODEBASE_MAP.md) | 目錄結構地圖、「我想改 X 要看哪裡」速查 |
| [`ARCHITECTURE.md`](ARCHITECTURE.md) | 系統架構、元件圖、設計決策 |
| [`DATA_MODEL.md`](DATA_MODEL.md) | Agent 資料模型、欄位說明、格式規範 |
| [`API_SURFACE.md`](API_SURFACE.md) | CLI 指令、參數、使用範例 |
| [`DEV_GUIDE.md`](DEV_GUIDE.md) | 開發環境建置、新增 agent 流程、貢獻指南 |
| [`DISCOVERY_LOG.md`](DISCOVERY_LOG.md) | 探索紀錄、技術債、待解問題 |

**中繼 context 檔案**（可刪除，供文件生成使用）：
- `.trace/_context/recon.md` — Stage 1 偵察報告
- `.trace/_context/web_findings.md` — 線上搜尋發現
- `.trace/_context/entry_points.md` — 入口點分析
- `.trace/_context/data_flow.md` — 資料流分析
- `.trace/_context/core_logic.md` — 核心邏輯分析
- `.trace/_context/extensions.md` — 擴充點分析
- `.trace/_context/integrations.md` — 整合分析
- `.trace/_context/configuration.md` — 設定與環境分析

---

## 專案專屬術語表

| 術語 | 說明 |
|------|------|
| **Agent** | 一個 `.md` 檔案，定義 AI 助手的 persona、mission、工作流程和成功指標 |
| **Division（分部）** | Agent 的業務類別分組（Engineering、Marketing 等），對應到 category 目錄 |
| **Frontmatter** | 每個 agent 檔案開頭的 YAML metadata block（`---` 包圍） |
| **SOUL** | Agent 的「人格」部分：Identity、Communication Style、Critical Rules |
| **AGENTS（Operations）** | Agent 的「能力」部分：Core Mission、Deliverables、Workflow、Metrics |
| **Slug** | Agent name 的 URL 友好格式，例如 `frontend-developer`（由 `slugify()` 生成） |
| **NEXUS** | Network of EXperts, Unified in Strategy — 多 agent 協調框架 |
| **NEXUS-Full/Sprint/Micro** | NEXUS 的三個執行模式，對應不同規模的工作範疇 |
| **Dev↔QA Loop** | NEXUS 框架中的開發品質控制循環，最多 3 次重試 |
| **Convert** | `convert.sh` 將 `.md` agent 轉換為工具特定格式的過程 |
| **Install** | `install.sh` 將轉換後的檔案部署到 AI 工具 config 目錄的過程 |
| **Accumulated format** | Aider/Windsurf 使用的單檔案格式，所有 agents 合併在一起 |
| **Sub-agent** | Claude Code 原生術語，指 `~/.claude/agents/` 中的 agent 定義 |
| **AGENT_DIRS** | 腳本中定義哪些目錄是 agent category 的核心陣列 |
| **MCP** | Model Context Protocol，用於為 agents 提供持久記憶的協議 |
| **vibe** | Agent frontmatter 中的一行個性描述，用於 OpenClaw IDENTITY.md |
