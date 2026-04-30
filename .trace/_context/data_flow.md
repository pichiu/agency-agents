# Stage 2.2 Data Flow

## 核心資料流：從 Agent 定義到 AI 工具安裝

這個專案的資料流是一個單向的 **transform-and-deploy pipeline**，沒有資料庫、HTTP 請求或狀態管理。

```
[.md 原始檔案] → [frontmatter 解析] → [格式轉換] → [整合檔案] → [安裝至工具]
```

## 完整 Use Case Trace：新增一個 Agent 並部署到 Claude Code

### 步驟 1：Agent 定義檔案（輸入）

原始來源：任何 category 目錄下的 `.md` 檔案，例如：

```
engineering/engineering-frontend-developer.md
```

檔案結構：
```
---                              ← frontmatter 開始標記
name: Frontend Developer
description: Expert frontend developer...
color: cyan
emoji: 🖥️
vibe: Builds responsive, accessible web apps...
---                              ← frontmatter 結束標記

# Frontend Developer Agent Personality
                                 ← body 開始
## 🧠 Your Identity & Memory
...
```

### 步驟 2：Frontmatter 解析

**函式**: `get_field()` (`scripts/convert.sh:87-93`)

```bash
get_field() {
  local field="$1" file="$2"
  awk -v f="$field" '
    /^---$/ { fm++; next }
    fm == 1 && $0 ~ "^" f ": " { sub("^" f ": ", ""); print; exit }
  ' "$file"
}
```

使用 AWK 解析：計數 `---` 出現次數，只在第一個 frontmatter block（`fm == 1`）中提取欄位值。

**函式**: `get_body()` (`scripts/convert.sh:96-99`)

```bash
get_body() {
  awk 'BEGIN{fm=0} /^---$/{fm++; next} fm>=2{print}' "$1"
}
```

當 `---` 出現第二次後，開始輸出所有後續內容（即 agent body）。

### 步驟 3：格式篩選（convert.sh `run_conversions()`）

**函式**: `run_conversions()` (`scripts/convert.sh:483-518`)

篩選邏輯（`scripts/convert.sh:491-495`）：
1. 只處理第一行是 `---` 的檔案（確認有 frontmatter）
2. 確認 `name` 欄位不為空（確認是 agent 定義）
3. 非 agent 文件（如 `examples/README.md`）被自動跳過

```bash
first_line="$(head -1 "$file")"
[[ "$first_line" == "---" ]] || continue  # 跳過無 frontmatter 的檔案
name="$(get_field "name" "$file")"
[[ -n "$name" ]] || continue              # 跳過無 name 的檔案
```

### 步驟 4：格式轉換（以 Claude Code 為例）

Claude Code 使用原始 `.md` 格式，**不需要轉換**。`install_claude_code()` 直接複製檔案：

```bash
# scripts/install.sh:305-319
install_claude_code() {
  local dest="${HOME}/.claude/agents"
  mkdir -p "$dest"
  for dir in "${AGENT_DIRS[@]}"; do
    while IFS= read -r -d '' f; do
      first_line="$(head -1 "$f")"
      [[ "$first_line" == "---" ]] || continue
      cp "$f" "$dest/"
    done < <(find "$REPO_ROOT/$dir" -name "*.md" -type f -print0)
  done
}
```

### 步驟 5：格式轉換（以 OpenClaw 為例，最複雜）

**函式**: `convert_openclaw()` (`scripts/convert.sh:251-341`)

OpenClaw 需要將每個 agent 分成三個獨立檔案：

```
SOUL.md    ← Identity, Communication, Critical Rules (persona sections)
AGENTS.md  ← Core Mission, Deliverables, Workflow, Metrics (operations sections)
IDENTITY.md ← emoji + name + vibe（一行摘要）
```

**分類邏輯**（`scripts/convert.sh:288-300`）：
```bash
# 逐行掃描 body，遇到 ## header 時根據關鍵字分類
if [[ "$header_lower" =~ identity ]] ||
   [[ "$header_lower" =~ learning.*memory ]] ||
   [[ "$header_lower" =~ communication ]] ||
   [[ "$header_lower" =~ style ]] ||
   [[ "$header_lower" =~ critical.rule ]] ||
   [[ "$header_lower" =~ rules.you.must.follow ]]; then
  current_target="soul"
else
  current_target="agents"
fi
```

這個邏輯與 `lint-agents.sh` 中的 `classify_header_target()` 函式完全相同（兩處需保持同步）。

### 步驟 6：格式轉換（以 Cursor 為例）

**函式**: `convert_cursor()` (`scripts/convert.sh:228-248`)

```
輸入: engineering-frontend-developer.md
輸出: integrations/cursor/rules/frontend-developer.mdc
```

Cursor `.mdc` 格式：
```
---
description: Expert frontend developer...
globs: ""
alwaysApply: false
---
(agent body)
```

### 步驟 7：名稱正規化（Slugify）

**函式**: `slugify()` (`scripts/convert.sh:103-105`)

```bash
slugify() {
  echo "$1" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//'
}
```

`"Frontend Developer"` → `"frontend-developer"`
`"AI Engineer"` → `"ai-engineer"`

Antigravity 格式前面再加 `agency-` 前綴：`"agency-frontend-developer"`

### 步驟 8：安裝到工具目錄

**函式**: `install_<tool>()` (`scripts/install.sh:305-519`)

安裝路徑彙整：

| 工具 | 範圍 | 安裝目標 |
|------|------|----------|
| Claude Code | 全域 | `~/.claude/agents/` |
| Copilot | 全域 | `~/.github/agents/` + `~/.copilot/agents/` |
| Antigravity | 全域 | `~/.gemini/antigravity/skills/agency-<slug>/` |
| Gemini CLI | 全域 | `~/.gemini/extensions/agency-agents/skills/<slug>/` |
| OpenClaw | 全域 | `~/.openclaw/agency-agents/<slug>/` |
| OpenCode | 專案 | `<cwd>/.opencode/agents/` |
| Cursor | 專案 | `<cwd>/.cursor/rules/` |
| Aider | 專案 | `<cwd>/CONVENTIONS.md` |
| Windsurf | 專案 | `<cwd>/.windsurfrules` |
| Qwen Code | 專案 | `<cwd>/.qwen/agents/` |
| Kimi Code | 全域 | `~/.config/kimi/agents/<slug>/` |

## 特殊資料流：Aider 與 Windsurf（Accumulation 模式）

這兩個工具只使用單一大檔案，所有 agent 都累積在裡面：

```
# scripts/convert.sh:411-479
AIDER_TMP="$(mktemp)"    # /tmp/tmp.XXXXX
WINDSURF_TMP="$(mktemp)" # /tmp/tmp.XXXXX

# 每個 agent 追加到暫時檔案
accumulate_aider()    # >> $AIDER_TMP
accumulate_windsurf() # >> $WINDSURF_TMP

# 全部處理完後，複製到最終位置
cp "$AIDER_TMP" "$OUT_DIR/aider/CONVENTIONS.md"
cp "$WINDSURF_TMP" "$OUT_DIR/windsurf/.windsurfrules"
```

## 並行模式資料流

當啟用 `--parallel` 時，使用 `xargs -P $N` 並行執行多個工具的轉換：

```bash
# scripts/convert.sh:576-578
printf '%s\n' "${parallel_tools[@]}" | \
  xargs -P "$parallel_jobs" -I {} \
  sh -c '"$AGENCY_CONVERT_SCRIPT" --tool "{}" --out "$AGENCY_CONVERT_OUT" > "$AGENCY_CONVERT_OUT_DIR/{}" 2>&1'
```

每個工具的輸出被緩衝到獨立的暫時檔案，處理完後再依序輸出，避免交錯。

注意：Aider 和 Windsurf 是 accumulation 模式（需要共享的暫時檔案），不能並行化，始終在串行模式執行（`scripts/convert.sh:582-591`）。

## 資料驗證流程（Lint）

```
[.md 檔案]
    │
    ▼
[first_line == "---"?] → NO → "ERROR: missing frontmatter"
    │ YES
    ▼
[frontmatter 有 name, description, color?] → NO → "ERROR: missing field"
    │ YES
    ▼
[body 有 Identity, Core Mission, Critical Rules?] → NO → "WARN: missing section"
    │
    ▼
[body 字數 >= 50?] → NO → "WARN: very short"
    │
    ▼
[有 SOUL headers AND AGENTS headers?] → NO → "WARN: no soul/agents sections"
    │
    ▼
PASSED
```
