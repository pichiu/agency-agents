# Stage 2.4 Extension Points

## 擴充機制概覽

這個專案有三個層次的擴充點，每個層次都有明確的接口定義：

## 層次 1：新增 Agent（最常見的貢獻方式）

**接口**：在任何 category 目錄中新增一個 `.md` 檔案，符合 agent template 格式即可自動被 toolchain 偵測和處理。

**必要條件**（`scripts/lint-agents.sh` 驗證）：
- 第一行必須是 `---`（frontmatter 標記）
- Frontmatter 必須包含：`name`, `description`, `color`
- Body 建議包含：`Identity`, `Core Mission`, `Critical Rules` sections

**推薦條件**（非強制，影響品質）：
- `emoji`, `vibe` 欄位（用於 OpenClaw IDENTITY.md）
- SOUL 類 sections（Identity, Communication, Critical Rules）
- AGENTS 類 sections（Core Mission, Deliverables, Workflow, Metrics）

**完全不需要**：
- 修改任何腳本
- 更新任何設定檔
- 執行任何構建步驟

**運作機制**：`run_conversions()` 使用 `find "$dirpath" -name "*.md" -type f` 動態掃描，新增的 `.md` 檔案自動被發現。

**可選欄位：`services`**

如果 agent 依賴外部服務，可在 frontmatter 聲明：

```yaml
services:
  - name: Service Name
    url: https://service-url.com
    tier: free  # free, freemium, or paid
```

⚠️ 注意：`services` 欄位目前不被 `convert.sh` 的任何轉換函式使用，僅作為文件性聲明（僅 `lint-agents.sh` 中的 `get_field` 可以讀取它，但不做任何處理）。

## 層次 2：新增工具支援（中等複雜度）

**接口**：在三個地方新增程式碼：

### 2a. `scripts/convert.sh` 中新增轉換函式

```bash
convert_newtool() {
  local file="$1"
  local name description slug outfile body
  
  name="$(get_field "name" "$file")"
  description="$(get_field "description" "$file")"
  slug="$(slugify "$name")"
  body="$(get_body "$file")"
  
  outfile="$OUT_DIR/newtool/<slug>.format"
  mkdir -p "$(dirname "$outfile")"
  
  cat > "$outfile" <<HEREDOC
  (tool-specific format using $name, $description, $body)
HEREDOC
}
```

### 2b. `scripts/install.sh` 中新增安裝函式和偵測函式

```bash
detect_newtool() { command -v newtool >/dev/null 2>&1 || [[ -d "${HOME}/.newtool" ]]; }

install_newtool() {
  local src="$INTEGRATIONS/newtool"
  local dest="${HOME}/.newtool/agents"
  mkdir -p "$dest"
  # 複製邏輯
}
```

### 2c. 更新三處陣列/case 語句

```bash
# convert.sh: valid_tools 陣列 + tools_to_run case + run_conversions case
# install.sh: ALL_TOOLS 陣列 + is_detected case + install_tool case + tool_label case
```

### 2d. 建立 `integrations/newtool/README.md`

說明工具的安裝步驟和使用方式。

**所有 Accumulated 格式的特殊情況**（如 Aider/Windsurf）：

這類工具需要額外的 accumulator 函式（`accumulate_<tool>()`），並在 `run_conversions()` 的 case 中調用。最後在 `main()` 中加入 flush 邏輯。

## 層次 3：新增 Agent Category（較少見）

**接口**：在 repo 根目錄新增一個目錄，並在**三個腳本**中同步更新 `AGENT_DIRS` 陣列。

```bash
# 需要同步修改的三個位置：
# scripts/convert.sh:64-67
# scripts/install.sh:107-110
# scripts/lint-agents.sh:12-26

AGENT_DIRS=(
  academic design engineering finance game-development marketing paid-media product project-management
  sales spatial-computing specialized strategy support testing
  new-category  # ← 新增這裡
)
```

⚠️ 這三處陣列必須保持完全一致，目前沒有 DRY 機制（是已知技術債）。

同時需要更新 `.github/workflows/lint-agents.yml` 的 `paths` 觸發條件：

```yaml
on:
  pull_request:
    paths:
      - "new-category/**"  # ← 新增
```

## 層次 4：新增 Game Engine 子目錄

`game-development/` 目錄有一個特殊的多層結構，包含引擎子目錄（`unity/`, `unreal-engine/`, `godot/`, `blender/`, `roblox-studio/`）。

**轉換腳本如何處理**：`find "$dirpath" -name "*.md" -type f` 是遞迴搜尋，所以子目錄中的 agents 自動被發現。這個模式可以在任何 category 中使用（但 `game-development/` 是唯一使用子目錄的 category）。

## MCP Memory 擴充點

`integrations/mcp-memory/` 提供了一個特殊的擴充模式：透過 MCP（Model Context Protocol）為 agents 提供持久記憶。

**接口**：`backend-architect-with-memory.md` 是一個示範性 agent，展示如何在 agent body 中使用 MCP memory 工具：

```markdown
# 使用 MCP memory 工具
- remember: 儲存決策和上下文
- recall: 搜尋過去的記憶
- rollback: 回滾到前一狀態
```

⚠️ `setup.sh` 只是一個骨架/佔位符，需要用戶自行選擇 MCP memory server 實作。

## 擴充點摘要

| 擴充目標 | 所需修改 | 難度 |
|----------|----------|------|
| 新增單個 agent | 新增 1 個 `.md` 文件 | ⭐ 極易 |
| 新增 agent category | 修改 3 個腳本 + CI config | ⭐⭐ 易 |
| 新增工具支援（per-agent 格式） | 修改 2 個腳本 + 新增 README | ⭐⭐⭐ 中 |
| 新增工具支援（accumulated 格式） | 修改 2 個腳本 + 新增 README | ⭐⭐⭐⭐ 中上 |
| 新增 GitHub Action 支援 | 新增 workflow + 可能的腳本 | ⭐⭐⭐ 中 |
