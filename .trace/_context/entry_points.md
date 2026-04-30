# Stage 2.1 Entry Points

## 概覽

這個專案沒有傳統的「程式啟動入口」（main function、HTTP server、CLI daemon）。它的「入口點」是三支 Bash 腳本，每支都是獨立的 CLI 工具。

## 主要入口點

### 1. `scripts/convert.sh` — Agent 格式轉換

**用途**: 將 `.md` agent 檔案轉換為各 AI 工具的特定格式
**啟動方式**:

```bash
./scripts/convert.sh                      # 轉換所有工具
./scripts/convert.sh --tool cursor        # 只轉換 Cursor 格式
./scripts/convert.sh --parallel           # 並行模式
./scripts/convert.sh --tool gemini-cli    # Gemini CLI 格式
```

**初始化流程** (`scripts/convert.sh:522-639`):

1. `main()` 解析命令列參數（`--tool`, `--out`, `--parallel`, `--jobs`）
2. 驗證 `$tool` 參數是否合法（`valid_tools` 陣列）
3. 初始化 `AIDER_TMP` 和 `WINDSURF_TMP` 暫時檔案（`mktemp`）
4. 決定是否啟用 parallel 模式（xargs + AGENCY_CONVERT_SCRIPT 環境變數）
5. 對每個工具呼叫 `run_conversions()` → 遍歷 `AGENT_DIRS` → 對每個 `.md` 檔案呼叫對應的 `convert_<tool>()` 函式
6. Aider/Windsurf 是 accumulation 模式（寫入暫時檔案，最後一次性輸出）

**全局變數**:
- `SCRIPT_DIR`, `REPO_ROOT`, `OUT_DIR` — 路徑相關
- `TODAY` — 用於 antigravity SKILL.md 的 `date_added` 欄位
- `AGENT_DIRS` — 固定陣列，定義哪些目錄是 agent category

### 2. `scripts/install.sh` — Agent 安裝

**用途**: 將 `integrations/` 中的轉換後檔案安裝到各工具的 config 目錄
**啟動方式**:

```bash
./scripts/install.sh                      # 互動模式（終端機下）
./scripts/install.sh --tool claude-code   # 安裝指定工具
./scripts/install.sh --no-interactive     # 非互動模式（CI 用）
./scripts/install.sh --parallel           # 並行安裝
```

**初始化流程** (`scripts/install.sh:540-664`):

1. `main()` 解析參數
2. `check_integrations()` — 確認 `integrations/` 目錄存在（`scripts/install.sh:131-136`）
3. 根據 `interactive_mode` 決定 UI 模式：
   - 互動模式：呼叫 `interactive_select()` 顯示 checkbox 選單
   - 非互動模式：`is_detected()` 自動掃描已安裝工具
4. 對每個選取的工具呼叫 `install_<tool>()` 函式

**工具偵測機制** (`scripts/install.sh:141-168`):
每個 `detect_<tool>()` 函式檢查：
- `command -v <tool>` — 是否可呼叫該 CLI
- 目錄存在性（例如 `~/.claude`, `~/.gemini/antigravity/skills`）

### 3. `scripts/lint-agents.sh` — Agent 格式驗證

**用途**: 驗證 agent `.md` 檔案是否符合規範
**啟動方式**:

```bash
./scripts/lint-agents.sh                  # 掃描所有 agent 目錄
./scripts/lint-agents.sh path/to/agent.md  # 只驗證指定檔案
```

**驗證流程** (`scripts/lint-agents.sh:59-109`):

1. 收集要 lint 的檔案（命令列參數 或 掃描 `AGENT_DIRS`）
2. 對每個檔案呼叫 `lint_file()`：
   a. 確認第一行是 `---`（frontmatter 開始）
   b. 提取 frontmatter 內容
   c. 檢查必填欄位：`name`, `description`, `color` → ERROR 若缺失
   d. 檢查推薦 sections：`Identity`, `Core Mission`, `Critical Rules` → WARN 若缺失
   e. 檢查 body 字數 < 50 → WARN
   f. 分析 `##` headers 是否有 SOUL 類和 AGENTS 類的分佈
3. 統計 errors 和 warnings，`exit 1` 若有 error

### 4. GitHub Actions CI — 自動化驗證

**檔案**: `.github/workflows/lint-agents.yml`

**觸發條件**: PR 中修改了 agent category 目錄下的任何 `.md` 檔案

**流程**:
1. `git diff` 取得 changed files
2. 呼叫 `./scripts/lint-agents.sh $CHANGED_FILES`
3. lint 失敗則 CI 失敗，阻止 merge

## 無程式碼啟動路徑

除了上述腳本，使用者也可以直接「手動」使用：
- 複製 agent `.md` 檔案到 AI 工具的 agents 目錄
- 無需任何腳本，因為 Claude Code 和 GitHub Copilot 原生支援 `.md` + frontmatter 格式

## AGENT_DIRS — 核心設定陣列

此陣列定義了哪些目錄被視為 agent category，出現在三支腳本中（必須保持同步）：

```bash
# scripts/convert.sh:64-67
# scripts/install.sh:107-110  
# scripts/lint-agents.sh:12-26
AGENT_DIRS=(
  academic design engineering finance game-development marketing paid-media product project-management
  sales spatial-computing specialized strategy support testing
)
```

注意：`strategy/` 在腳本中存在但不在 README 的分部清單中，是一個隱性的 undocumented 分部。
