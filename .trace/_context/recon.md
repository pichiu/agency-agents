# Stage 1 偵察報告 (Reconnaissance)

## 1.1 專案基本資訊

**專案名稱**: The Agency — AI Specialists Ready to Transform Your Workflow
**GitHub**: https://github.com/msitarzewski/agency-agents
**維護者**: Michael Sitarzewski (@msitarzewski)
**授權**: MIT License
**起源**: 由一篇 Reddit 討論串（r/ClaudeAI）演化而來，在 12 小時內收到 50+ 個請求

## 1.2 技術棧識別

這個專案**沒有任何傳統意義上的程式語言** — 沒有 JavaScript/TypeScript、Python、Go 等後端或前端框架。技術棧如下：

| 類別 | 技術 | 用途 |
|------|------|------|
| 文件格式 | Markdown (.md) with YAML frontmatter | Agent 定義檔案 |
| 腳本語言 | Bash | convert.sh、install.sh、lint-agents.sh |
| CI/CD | GitHub Actions | PR 時自動 lint agent 檔案 |
| 設定格式 | YAML | frontmatter 解析、Kimi agent.yaml |
| 國際化腳本 | PowerShell (.ps1) | Windows 環境下的 zh-CN 本地化 |
| i18n 資料 | JSON | agent-names-zh.json (名稱對照表) |

## 1.3 目錄結構（3 層深度）

```
agency-agents/
├── README.md                    # 主文件：完整的 agent roster + 使用說明
├── CONTRIBUTING.md              # 貢獻指南：agent 設計規範、PR 流程
├── CONTRIBUTING_zh-CN.md        # 簡體中文貢獻指南
├── SECURITY.md                  # 安全政策
├── LICENSE                      # MIT License
├── .gitignore                   # 排除 generated 整合檔案
├── .gitattributes               # Git 屬性設定
│
├── academic/                    # 學術分部（5 個 agents）
│   ├── academic-anthropologist.md
│   ├── academic-geographer.md
│   ├── academic-historian.md
│   ├── academic-narratologist.md
│   └── academic-psychologist.md
│
├── design/                      # 設計分部（8 個 agents）
│   ├── design-brand-guardian.md
│   ├── design-image-prompt-engineer.md
│   ├── design-inclusive-visuals-specialist.md
│   ├── design-ui-designer.md
│   ├── design-ux-architect.md
│   ├── design-ux-researcher.md
│   ├── design-visual-storyteller.md
│   └── design-whimsy-injector.md
│
├── engineering/                 # 工程分部（23 個 agents）
│   ├── engineering-ai-engineer.md
│   ├── engineering-backend-architect.md
│   ├── engineering-codebase-onboarding-engineer.md
│   ├── engineering-code-reviewer.md
│   ├── engineering-data-engineer.md
│   ├── ... (共 23 個)
│
├── finance/                     # 財務分部（5 個 agents）
├── game-development/            # 遊戲開發分部（含子目錄）
│   ├── unity/                   # Unity 相關 agents
│   ├── unreal-engine/           # Unreal Engine 相關 agents
│   ├── godot/                   # Godot 相關 agents
│   ├── blender/                 # Blender 相關 agents
│   └── roblox-studio/           # Roblox Studio 相關 agents
├── marketing/                   # 行銷分部（25+ 個 agents）
├── paid-media/                  # 付費媒體分部（7 個 agents）
├── product/                     # 產品分部（5 個 agents）
├── project-management/          # 專案管理分部（6 個 agents）
├── sales/                       # 銷售分部（8 個 agents）
├── spatial-computing/           # 空間運算分部（6 個 agents）
├── specialized/                 # 特殊分部（40+ 個 agents）
├── strategy/                    # 策略分部（含子目錄）
│   ├── coordination/
│   ├── playbooks/
│   └── runbooks/
├── support/                     # 支援分部（6 個 agents）
├── testing/                     # 測試分部（8 個 agents）
│
├── scripts/                     # 工具腳本
│   ├── convert.sh               # 將 .md agents 轉換為各工具格式
│   ├── install.sh               # 安裝 agents 至各 AI 工具
│   ├── lint-agents.sh           # 驗證 agent 檔案格式
│   └── i18n/
│       ├── agent-names-zh.json  # 中文名稱對照表
│       └── localize-agents-zh.ps1  # Windows PowerShell 本地化腳本
│
├── integrations/                # 整合格式目錄（大部分由 convert.sh 生成，git ignored）
│   ├── claude-code/README.md    # Claude Code 整合說明
│   ├── github-copilot/README.md
│   ├── antigravity/README.md
│   ├── gemini-cli/README.md
│   ├── opencode/README.md
│   ├── cursor/README.md
│   ├── aider/README.md
│   ├── windsurf/README.md
│   ├── openclaw/README.md
│   ├── kimi/README.md
│   ├── qwen/README.md
│   └── mcp-memory/              # MCP 記憶整合（setup.sh + README + 範例 agent）
│
├── examples/                    # 多 agent 協作範例
│   ├── README.md
│   ├── nexus-spatial-discovery.md   # 8 個 agents 同時協作的完整範例
│   ├── workflow-book-chapter.md
│   ├── workflow-landing-page.md
│   ├── workflow-startup-mvp.md
│   └── workflow-with-memory.md
│
└── .github/                     # GitHub 設定
    ├── workflows/
    │   └── lint-agents.yml      # CI: PR 時自動 lint
    ├── ISSUE_TEMPLATE/
    │   ├── bug-report.yml
    │   └── new-agent-request.yml
    ├── PULL_REQUEST_TEMPLATE.md
    └── FUNDING.yml
```

## 1.4 架構模式

這是一個 **Content Repository + Toolchain** 架構（非傳統軟體架構）：

- **核心資產**: Markdown 格式的 agent 定義檔案（YAML frontmatter + Markdown body）
- **轉換層**: `convert.sh` 將通用格式轉為工具特定格式（antigravity、cursor、aider 等）
- **安裝層**: `install.sh` 將轉換後的檔案部署到各 AI 工具的 config 目錄
- **品質控制**: `lint-agents.sh` + GitHub Actions CI 驗證 agent 格式正確性

## 1.5 Agent 檔案格式

每個 agent 檔案結構如下：

```markdown
---
name: Agent Name
description: One-line description
color: cyan                  # 顏色名稱或 hex code
emoji: 🎯                   # 可選
vibe: One-line personality  # 可選，用於 OpenClaw IDENTITY.md
services:                    # 可選，只有依賴外部服務時才填
  - name: Service Name
    url: https://...
    tier: free
---

# Agent Name

## 🧠 Your Identity & Memory    ← 分類為 SOUL (persona)
## 🎯 Your Core Mission         ← 分類為 AGENTS (operations)
## 🚨 Critical Rules            ← 分類為 SOUL (persona)
## 📋 Your Technical Deliverables ← AGENTS
## 🔄 Your Workflow Process     ← AGENTS
## 💭 Your Communication Style  ← SOUL
## 🔄 Learning & Memory         ← SOUL
## 🎯 Your Success Metrics      ← AGENTS
## 🚀 Advanced Capabilities     ← AGENTS
```

**Frontmatter 必填欄位**: `name`, `description`, `color`
**推薦 section**: `Identity`, `Core Mission`, `Critical Rules`

## 1.6 Agent 分部與數量統計

| 分部 | 目錄 | Agent 數量 |
|------|------|------------|
| Engineering | `engineering/` | 23 |
| Marketing | `marketing/` | 25+ |
| Specialized | `specialized/` | 40+ |
| Testing | `testing/` | 8 |
| Design | `design/` | 8 |
| Sales | `sales/` | 8 |
| Support | `support/` | 6 |
| Project Management | `project-management/` | 6 |
| Finance | `finance/` | 5 |
| Academic | `academic/` | 5 |
| Product | `product/` | 5 |
| Paid Media | `paid-media/` | 7 |
| Spatial Computing | `spatial-computing/` | 6 |
| Game Development | `game-development/` | 17+ |
| Strategy | `strategy/` | 未知（含子目錄） |
| **合計** | | **約 144-147 個** |

## 1.7 支援的 AI 工具平台

| 工具 | 格式 | 安裝位置 |
|------|------|----------|
| Claude Code | `.md` (原生，無需轉換) | `~/.claude/agents/` |
| GitHub Copilot | `.md` (原生) | `~/.github/agents/` + `~/.copilot/agents/` |
| Antigravity (Gemini) | `SKILL.md` per agent | `~/.gemini/antigravity/skills/agency-<slug>/` |
| Gemini CLI | `SKILL.md` + extension manifest | `~/.gemini/extensions/agency-agents/` |
| OpenCode | `.md` with mode:subagent | `.opencode/agents/` |
| Cursor | `.mdc` rule files | `.cursor/rules/` |
| Aider | 單一 `CONVENTIONS.md` | `./CONVENTIONS.md` |
| Windsurf | 單一 `.windsurfrules` | `./.windsurfrules` |
| OpenClaw | `SOUL.md` + `AGENTS.md` + `IDENTITY.md` | `~/.openclaw/agency-agents/` |
| Qwen Code | `.md` SubAgent files | `~/.qwen/agents/` |
| Kimi Code | `agent.yaml` + `system.md` | `~/.config/kimi/agents/` |

## 1.8 既有文件盤點

下列為 repo 中既有的文件資產：

| 檔案/目錄 | 類型 | 說明 |
|-----------|------|------|
| `README.md` | 主文件 | 完整的 agent roster、使用方式、整合說明 |
| `CONTRIBUTING.md` | 貢獻指南 | agent 設計規範、PR 流程、風格指南 |
| `CONTRIBUTING_zh-CN.md` | 翻譯 | 簡體中文貢獻指南 |
| `integrations/*/README.md` | 整合說明 | 各工具的安裝與使用方式 |
| `examples/*.md` | 範例 | 多 agent 協作工作流程範例 |
| `strategy/` | 策略文件 | EXECUTIVE-BRIEF.md、QUICKSTART.md、coordination/、playbooks/、runbooks/ |

## 1.9 文件與程式碼之間的落差分析

1. **README 說 144 個 agents，實際程式碼中聲稱 147 個**: README 底部致謝區塊說「147 agents across 12 divisions」，但 Stats 區塊說「144 Specialized Agents」。程式碼中實際有約 226 個 .md 檔案，扣除 README/CONTRIBUTING 等非 agent 文件後約為 144-147 個。

2. **`strategy/` 目錄存在但 README 未列出**: CONTRIBUTING.md 的分部清單未包含 `strategy/`，但 `AGENT_DIRS` 在腳本中有包含它（`scripts/lint-agents.sh` 第 20 行）。

3. **`integrations/` 大部分是 gitignored 的生成檔案**: `.gitignore` 明確排除了所有由 `convert.sh` 生成的檔案，只保留 README.md 和 setup.sh。新 clone 的 repo 中 `integrations/` 幾乎是空的（只有 READMEs），需先執行 `convert.sh` 才能使用大多數工具。

4. **MCP memory integration 是佔位符**: `integrations/mcp-memory/setup.sh` 未指定具體的 MCP memory server，要求用戶自行選擇，實際功能未完全實作。

## 1.10 檔案與程式碼行數

- 總檔案數：239 個（含 .git）
- Markdown 檔案：226 個
- 非 Markdown 檔案：13 個（腳本、YAML、JSON、PowerShell）
- 無傳統程式語言（Python/JS/TS/Go 等）的原始碼
