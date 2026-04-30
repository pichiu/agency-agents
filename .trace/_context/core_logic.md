# Stage 2.3 核心領域邏輯

## 什麼是這個專案的「心臟」

這個專案的核心不是演算法，而是**兩個緊密相連的設計決策**：

1. **Agent 定義格式** — YAML frontmatter + Markdown body 的雙層結構
2. **Persona/Operations 分割邏輯** — 將 agent 的「是誰」與「做什麼」分開

## 核心 Abstraction 1：Agent 定義格式

每個 agent 檔案是一個「二合一」文件：

```
┌──────────────────────────────────────────┐
│  YAML Frontmatter (機器可讀 metadata)    │
│  name, description, color, emoji, vibe   │
├──────────────────────────────────────────┤
│  Markdown Body (人類可讀 + AI 可讀)       │
│  ## Identity & Memory                    │
│  ## Core Mission                         │
│  ## Critical Rules                       │
│  ## Technical Deliverables               │
│  ## Workflow Process                     │
│  ## Communication Style                  │
│  ## Learning & Memory                    │
│  ## Success Metrics                      │
│  ## Advanced Capabilities                │
└──────────────────────────────────────────┘
```

**設計哲學**：frontmatter 提供結構化 metadata 讓工具可以程式化處理（顏色、slug、排序），body 是純文字的 system prompt，讓人類可讀、可修改、可版本控制。

## 核心 Abstraction 2：Persona / Operations 分割

這是整個 codebase 中唯一真正複雜的邏輯——將 agent body 分成兩個語義部分。

**實作位置**：
- `scripts/convert.sh:286-313` — `convert_openclaw()` 中的分類邏輯
- `scripts/lint-agents.sh:38-53` — `classify_header_target()` 的鏡像實作

**SOUL（persona）— 誰是這個 agent**：
```
包含這些關鍵字的 ## headers → SOUL.md
- identity
- learning.*memory (regex)
- communication
- style
- critical.rule (regex)
- rules.you.must.follow (regex)
```

**AGENTS（operations）— agent 做什麼**：
```
其他所有 ## headers → AGENTS.md
- core mission
- technical deliverables
- workflow process
- success metrics
- advanced capabilities
```

**為什麼這樣設計**：這個分割讓 OpenClaw（以及其他可能的工具）可以獨立存取 agent 的「人格」和「能力」，支援更細粒度的 agent 組合和重用。

## 核心函式分析

### `get_field()` — Frontmatter 解析

```bash
# scripts/convert.sh:87-93
get_field() {
  local field="$1" file="$2"
  awk -v f="$field" '
    /^---$/ { fm++; next }
    fm == 1 && $0 ~ "^" f ": " { sub("^" f ": ", ""); print; exit }
  ' "$file"
}
```

**邏輯**：
- AWK 計數 `---` 的出現次數（`fm` = frontmatter counter）
- 只在第一個 block（`fm == 1`，介於第 1 和第 2 個 `---` 之間）中查找欄位
- 找到後立即 `exit`（不繼續掃描 body）

**Limitation**：只支援 flat key: value 格式，不支援 YAML 嵌套（像 `services:` 的多行 block 需要特殊處理）。services 欄位本身被 lint 通過但不被任何轉換腳本使用（⚠️ 待確認）。

### `get_body()` — Body 提取

```bash
# scripts/convert.sh:96-99
get_body() {
  awk 'BEGIN{fm=0} /^---$/{fm++; next} fm>=2{print}' "$1"
}
```

**邏輯**：當第二個 `---` 出現後（`fm >= 2`），印出所有後續行。簡潔但有一個邊界條件：如果 body 中也有 `---` 分隔線，它會被忽略（因為 AWK 條件是 `fm >= 2`，遇到第三個 `---` 後 `fm` 變成 3，仍然 `>= 2`，繼續印出）。

### `slugify()` — 名稱正規化

```bash
# scripts/convert.sh:103-105
slugify() {
  echo "$1" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//'
}
```

**管道**：
1. 轉小寫
2. 非字母數字字元替換為 `-`
3. 多個連續 `-` 合併為一個
4. 移除首尾的 `-`

**範例**：`"AI Engineer"` → `"ai-engineer"`, `"FP&A Analyst"` → `"fp-a-analyst"`

### `resolve_opencode_color()` — 顏色正規化

```bash
# scripts/convert.sh:158-200
```

**邏輯**：將 agent frontmatter 中的顏色名稱（`cyan`, `blue`, `purple` 等）對應到 `#RRGGBB` hex 格式。OpenCode 只接受 hex 格式，所以需要一個預定義的映射表。未知顏色回退到灰色 `#6B7280`。

## 設計模式分析

### 1. Pipeline Pattern

整個系統是典型的 Unix pipe 設計：

```
find (agent files)
  → head -1 (filter: has frontmatter)
  → get_field (extract metadata)
  → get_body (extract content)
  → convert_<tool> (transform)
  → write output file
```

### 2. Strategy Pattern（工具轉換器）

每個工具有獨立的 `convert_<tool>()` 函式，統一介面：接受一個 `$file` 參數，輸出到 `$OUT_DIR/<tool>/`。`run_conversions()` 用 case 語句根據 tool 名稱 dispatch 到對應函式。

### 3. Accumulator Pattern（Aider/Windsurf）

不同於其他「一個 agent = 一個輸出檔案」的策略，Aider 和 Windsurf 使用 accumulator pattern：所有 agents 寫入同一個 temp file，最後 flush 到最終位置。

### 4. Environment Variable IPC（並行模式）

並行執行時，父程序透過環境變數傳遞資訊給子程序：
- `AGENCY_INSTALL_WORKER=1` — 標記當前是 worker 子程序，跳過 header/footer 輸出
- `AGENCY_INSTALL_OUT_DIR` — 輸出緩衝目錄
- `AGENCY_CONVERT_SCRIPT` / `AGENCY_INSTALL_SCRIPT` — 腳本自身路徑

## Agent Body 設計哲學（非程式碼層面的核心）

雖然不是程式碼，但 agent body 的設計是這個專案最核心的知識資產：

**每個 agent 必須有**：
1. **強烈個性** — 不是「我是一個有用的助手」，而是具體的聲音（例如「I default to finding 3-5 issues and require visual proof」）
2. **可量化的成功指標** — 例如「Page load times under 3 seconds on 3G」
3. **實際可運行的程式碼範例** — 不是偽碼
4. **逐步工作流程** — 真實世界測試過的流程

**NEXUS 模式（meta-agent 協調）**：
`strategy/` 目錄定義了 NEXUS（Network of EXperts, Unified in Strategy）框架，將多個 agents 組織成協調管線：
- PM → Architect/UX → [Dev ↔ QA Loop（最多 3 次重試）] → Integration
- 三個模式：Full（12-24 週）、Sprint（2-6 週）、Micro（1-5 天）
- `specialized/agents-orchestrator.md` 是這個框架的指揮 agent
