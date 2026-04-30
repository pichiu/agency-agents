# Stage 2.5 外部整合

## 概覽

這個專案**沒有任何外部 API 呼叫或網路請求**。所有「整合」都是本地端的檔案系統操作：將格式化的 prompt 文件複製到各 AI 工具的 config 目錄中。

真正的外部依賴只有一個：**用戶已安裝的 AI 工具**。

## 整合目標（Downstream 工具）

### Claude Code（原生整合，最優先）

- **整合方式**: 直接複製 `.md` 檔案（無需轉換）
- **原生格式**: `.md` + YAML frontmatter 就是 Claude Code sub-agent 格式
- **安裝路徑**: `~/.claude/agents/`
- **使用方式**: 在 Claude Code 對話中直接呼叫 agent 名稱
- **文件**: `integrations/claude-code/README.md`

### GitHub Copilot（原生整合）

- **整合方式**: 直接複製 `.md` 檔案（無需轉換）
- **安裝路徑**: `~/.github/agents/` 和 `~/.copilot/agents/`
- **特殊要求**: VS Code 設定 `chat.agentFilesLocations` 需要指向安裝路徑
- **文件**: `integrations/github-copilot/README.md`

### Antigravity（Gemini 擴充）

- **整合方式**: `convert_antigravity()` 轉換
- **輸出格式**: `integrations/antigravity/agency-<slug>/SKILL.md`
- **SKILL.md 格式**：
  ```yaml
  ---
  name: agency-<slug>
  description: <description>
  risk: low
  source: community
  date_added: 'YYYY-MM-DD'
  ---
  <agent body>
  ```
- **安裝路徑**: `~/.gemini/antigravity/skills/agency-<slug>/`
- **使用方式**: `@agency-frontend-developer review this React component`

### Gemini CLI（Extension 格式）

- **整合方式**: `convert_gemini_cli()` 轉換 + 自動生成 manifest
- **輸出格式**: `integrations/gemini-cli/skills/<slug>/SKILL.md` + `gemini-extension.json`
- **Manifest** (`gemini-extension.json`):
  ```json
  {
    "name": "agency-agents",
    "version": "1.0.0"
  }
  ```
- **安裝路徑**: `~/.gemini/extensions/agency-agents/`

### OpenCode（Sub-agent 格式）

- **整合方式**: `convert_opencode()` 轉換
- **輸出格式**: `integrations/opencode/agents/<slug>.md`
- **特殊欄位**: 加入 `mode: subagent` 和 OpenCode hex 顏色
- **安裝路徑**: `.opencode/agents/`（專案範圍）或 `~/.config/opencode/agents/`（全局）
- **顏色轉換**: `resolve_opencode_color()` 將顏色名稱正規化為 `#RRGGBB`

### Cursor（Rules 格式）

- **整合方式**: `convert_cursor()` 轉換
- **輸出格式**: `integrations/cursor/rules/<slug>.mdc`
- **`.mdc` 格式**：
  ```yaml
  ---
  description: <description>
  globs: ""
  alwaysApply: false
  ---
  <agent body>
  ```
- **安裝路徑**: `.cursor/rules/`（專案範圍）
- **使用方式**: `Use the @security-engineer rules to review this code`

### Aider（單一 CONVENTIONS.md）

- **整合方式**: `accumulate_aider()` — 所有 agents 累積到一個檔案
- **輸出格式**: `integrations/aider/CONVENTIONS.md`
- **格式**：
  ```markdown
  ---
  ## Agent Name
  > description
  <agent body>
  ```
- **安裝路徑**: `./CONVENTIONS.md`（專案根目錄）
- **使用方式**: `Use the Frontend Developer agent to refactor this component`

### Windsurf（單一 .windsurfrules）

- **整合方式**: `accumulate_windsurf()` — 所有 agents 累積到一個檔案
- **輸出格式**: `integrations/windsurf/.windsurfrules`
- **格式**：使用 `================================================================================` 作為分隔符
- **安裝路徑**: `./.windsurfrules`（專案根目錄）

### OpenClaw（Workspace 格式）

- **整合方式**: `convert_openclaw()` — 最複雜的轉換
- **輸出格式**: `integrations/openclaw/<slug>/` 目錄，包含三個檔案：
  - `SOUL.md` — persona sections（Identity, Communication, Critical Rules）
  - `AGENTS.md` — operations sections（Mission, Deliverables, Workflow）
  - `IDENTITY.md` — `# {emoji} {name}\n{vibe}` 一行摘要
- **安裝路徑**: `~/.openclaw/agency-agents/<slug>/`
- **CLI 整合**: 如果 `openclaw` CLI 可用，`install_openclaw()` 會自動執行 `openclaw agents add` 命令
- **後處理**: 安裝後需執行 `openclaw gateway restart` 才能啟用

### Qwen Code（SubAgent 格式）

- **整合方式**: `convert_qwen()` 轉換
- **輸出格式**: `integrations/qwen/agents/<slug>.md`
- **特殊支援**: `tools` 欄位（若 agent frontmatter 有 `tools:` 則繼承）
- **特殊支援**: `${variable}` 模板變數（agent body 中可用）
- **安裝路徑**: `.qwen/agents/`（專案範圍）

### Kimi Code（YAML + System Prompt 格式）

- **整合方式**: `convert_kimi()` 轉換
- **輸出格式**: `integrations/kimi/<slug>/` 目錄，包含：
  - `agent.yaml` — 設定檔（name, extend: default, system_prompt_path）
  - `system.md` — system prompt 完整內容
- **安裝路徑**: `~/.config/kimi/agents/<slug>/`
- **使用方式**:
  ```bash
  kimi --agent-file ~/.config/kimi/agents/<slug>/agent.yaml
  ```

## 外部工具偵測邏輯

`install.sh` 中每個工具有獨立的偵測函式，偵測策略：

| 工具 | 偵測方法 |
|------|----------|
| claude-code | `[[ -d ~/.claude ]]` |
| copilot | `command -v code` OR `~/.github` OR `~/.copilot` |
| antigravity | `[[ -d ~/.gemini/antigravity/skills ]]` |
| gemini-cli | `command -v gemini` OR `[[ -d ~/.gemini ]]` |
| cursor | `command -v cursor` OR `[[ -d ~/.cursor ]]` |
| opencode | `command -v opencode` OR `~/.config/opencode` |
| aider | `command -v aider` |
| openclaw | `command -v openclaw` OR `[[ -d ~/.openclaw ]]` |
| windsurf | `command -v windsurf` OR `~/.codeium` |
| qwen | `command -v qwen` OR `[[ -d ~/.qwen ]]` |
| kimi | `command -v kimi` |

## 失敗處理

- **convert.sh**: 如果 `integrations/` 目錄不存在，`install.sh` 會顯示 error 並退出（`check_integrations()`）
- **各工具 installer**: 如果 `integrations/<tool>/` 目錄不存在，顯示 error 並 `return 1`
- **OpenClaw CLI**: 如果 `openclaw agents add` 失敗，使用 `|| true` 忽略錯誤（不中斷安裝）
- **Windsurf/Aider 重複安裝**: 偵測到目標檔案已存在時，顯示 warn 並跳過（不覆蓋）

## 生成檔案管理（.gitignore）

絕大多數 `integrations/` 下的生成檔案被 `.gitignore` 排除：

```
integrations/antigravity/agency-*/
integrations/gemini-cli/skills/
integrations/cursor/rules/
integrations/aider/CONVENTIONS.md
integrations/windsurf/.windsurfrules
integrations/openclaw/* (但 README.md 除外)
integrations/qwen/agents/
integrations/kimi/*/ (但 README.md 除外)
```

**原因**：
1. 生成檔案可從源文件重新生成，不需要版本控制
2. 避免在合並時產生大量衝突
3. 讓 repo 保持輕量（只追蹤源文件）

**保留在 git 中的例外**：
- `integrations/*/README.md` — 文件
- `integrations/mcp-memory/setup.sh` — 非生成的手工腳本
- `integrations/claude-code/` 和 `integrations/github-copilot/` 整個目錄 — 因為這兩個工具直接使用源 `.md` 格式
