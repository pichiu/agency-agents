# Stage 2.6 設定與環境

## 概覽

這個專案**沒有環境變數設定、沒有設定檔、沒有 secrets 管理**。所有設定都是：
1. 硬編碼在腳本中的常數（`AGENT_DIRS`、`ALL_TOOLS`）
2. 命令列參數（`--tool`、`--parallel`、`--jobs`）
3. 工具偵測時自動推斷的路徑（`$HOME/.claude`, `$HOME/.cursor` 等）

## 命令列參數設定

### `scripts/convert.sh`

| 參數 | 預設值 | 說明 |
|------|--------|------|
| `--tool <name>` | `all` | 只轉換指定工具的格式 |
| `--out <dir>` | `$REPO_ROOT/integrations` | 輸出目錄 |
| `--parallel` | false | 並行模式（多工具同時處理） |
| `--jobs N` | `nproc` 或 `sysctl hw.ncpu` 或 4 | 並行 job 數量 |

### `scripts/install.sh`

| 參數 | 預設值 | 說明 |
|------|--------|------|
| `--tool <name>` | `all` | 只安裝指定工具 |
| `--interactive` | auto | 強制互動模式 |
| `--no-interactive` | auto | 強制非互動模式（CI 用） |
| `--parallel` | false | 並行安裝模式 |
| `--jobs N` | `nproc` 或 4 | 並行 job 數量 |

**互動模式自動判斷**（`scripts/install.sh:572-576`）：

```bash
if [[ "$interactive_mode" == "auto" && -t 0 && -t 1 && "$tool" == "all" ]]; then
  use_interactive=true
fi
```

條件：同時在終端機輸入/輸出（`-t 0 && -t 1`）且沒有指定具體工具。

### `scripts/lint-agents.sh`

| 參數 | 說明 |
|------|------|
| `[file ...]` | 可選，要 lint 的特定檔案清單；省略則掃描所有 AGENT_DIRS |

## 環境變數（內部使用）

這些環境變數只在並行模式中由腳本自身設置，**不需要用戶配置**：

| 環境變數 | 設置位置 | 用途 |
|----------|----------|------|
| `AGENCY_INSTALL_WORKER=1` | `install.sh` 並行 worker | 標記為子 worker，跳過 header/footer 輸出 |
| `AGENCY_INSTALL_OUT_DIR` | `install.sh` 主程序 | 並行 worker 的輸出緩衝目錄 |
| `AGENCY_INSTALL_SCRIPT` | `install.sh` 主程序 | 腳本自身路徑（給 xargs 使用） |
| `AGENCY_CONVERT_OUT_DIR` | `convert.sh` 主程序 | 並行 worker 的輸出緩衝目錄 |
| `AGENCY_CONVERT_SCRIPT` | `convert.sh` 主程序 | 腳本自身路徑（給 xargs 使用） |
| `AGENCY_CONVERT_OUT` | `convert.sh` 主程序 | 輸出目錄路徑（傳遞給 worker） |

## 顯示設定

顏色輸出由以下條件控制（`install.sh:41-51`, `convert.sh:33-37`）：

```bash
if [[ -t 1 && -z "${NO_COLOR:-}" && "${TERM:-}" != "dumb" ]]; then
  # 啟用彩色輸出
else
  # 純文字（CI/重定向場景）
fi
```

| 環境變數 | 說明 |
|----------|------|
| `NO_COLOR` | 設置任意值即停用顏色 |
| `TERM=dumb` | 停用顏色 |

## 硬編碼設定

### `AGENT_DIRS` 陣列

定義哪些目錄是 agent category，**三個腳本**中必須保持同步：

```bash
# scripts/convert.sh:64-67
# scripts/install.sh:107-110
# scripts/lint-agents.sh:12-26
AGENT_DIRS=(
  academic design engineering finance game-development marketing paid-media product project-management
  sales spatial-computing specialized strategy support testing
)
```

### `ALL_TOOLS` 陣列（`install.sh` 專用）

```bash
# scripts/install.sh:104
ALL_TOOLS=(claude-code copilot antigravity gemini-cli opencode openclaw cursor aider windsurf qwen kimi)
```

### `REQUIRED_FRONTMATTER` + `RECOMMENDED_SECTIONS`（`lint-agents.sh` 專用）

```bash
# scripts/lint-agents.sh:30-31
REQUIRED_FRONTMATTER=("name" "description" "color")
RECOMMENDED_SECTIONS=("Identity" "Core Mission" "Critical Rules")
```

### 顏色名稱映射表（`convert.sh` 專用）

`resolve_opencode_color()` 函式中有約 20 個顏色名稱到 hex 的映射，硬編碼在腳本中（`scripts/convert.sh:158-200`）。

### Gemini CLI Extension Manifest

Gemini CLI 的 `gemini-extension.json` 是 inline here-doc，硬編碼在 `convert.sh` 中：

```json
{
  "name": "agency-agents",
  "version": "1.0.0"
}
```

（`scripts/convert.sh:606-610`）

## Lint 設定（CI 行為）

GitHub Actions CI 設定（`.github/workflows/lint-agents.yml`）：

| 設定 | 值 |
|------|-----|
| 觸發條件 | PR 修改了 agent category 目錄下的 `.md` 檔案 |
| 執行環境 | `ubuntu-latest` |
| 差異基準 | `origin/${{ github.base_ref }}` |
| diff 篩選 | `ACMR`（Added, Copied, Modified, Renamed） |
| lint 目標 | 只 lint PR 中**被修改**的 agent 檔案（非全部） |

## i18n 設定（PowerShell 腳本）

`scripts/i18n/localize-agents-zh.ps1` 有一個可設定的參數：

```powershell
param(
  [string[]]$TargetDirs = @(
    "$env:USERPROFILE\.github\agents",
    "$env:USERPROFILE\.copilot\agents"
  )
)
```

修改 `$TargetDirs` 即可指向不同的安裝目錄。映射資料在 `scripts/i18n/agent-names-zh.json`。

## 無設定的設計哲學

這個專案刻意選擇不使用設定檔或環境變數，原因：

1. **降低使用門檻**：只需執行一個命令，無需配置任何東西
2. **避免配置漂移**：硬編碼常數保證所有用戶的行為一致
3. **符合 Unix 工具哲學**：小而專注，做好一件事

**Trade-off**：腳本的設定（如 `AGENT_DIRS`）需要在三個地方同步修改，這是已知的技術債。若未來新增 category 忘記同步，lint/convert/install 的行為會不一致。
