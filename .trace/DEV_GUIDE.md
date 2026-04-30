# The Agency — 開發者上手指南

> 快速掌握 agency-agents 專案的架構、開發流程與貢獻方法。

---

## 1. 環境需求（Prerequisites）

### 必要工具

| 工具 | 最低版本 | 說明 |
|------|----------|------|
| Bash | 3.2+ | 執行所有腳本（convert.sh、install.sh、lint-agents.sh） |
| git | 任意現代版本 | clone repo、提交 PR |

> 不需要 Node.js、Python、Docker 或任何語言執行環境。這個專案的核心資產是 Markdown 檔案。

### 支援的 AI 工具

| 工具 | 備註 |
|------|------|
| **Claude Code** | 原生支援，無需轉換 |
| **GitHub Copilot** | 原生支援，無需轉換 |
| **Cursor** | 需執行 `convert.sh --tool cursor` 產生 `.mdc` 格式 |
| **Aider** | 需執行 `convert.sh --tool aider` 產生單一 `CONVENTIONS.md` |
| **Windsurf** | 需執行 `convert.sh --tool windsurf` |
| **Gemini CLI / Antigravity** | 需執行 `convert.sh --tool gemini-cli` 或 `--tool antigravity` |
| **OpenCode / OpenClaw / Qwen / Kimi** | 各自有對應的轉換格式 |

### 平台支援

- **Linux**：完整支援（CI 環境為 `ubuntu-latest`）
- **macOS**：完整支援（Bash 3.2+ 已預裝）
- **Windows**：透過 Git Bash 或 WSL 執行腳本；i18n 中文化有額外提供 PowerShell 腳本（`scripts/i18n/localize-agents-zh.ps1`）

---

## 2. 快速開始（5 分鐘）

### Clone 並安裝

```bash
git clone https://github.com/msitarzewski/agency-agents
cd agency-agents

# 方式一：直接複製給 Claude Code（最快）
cp engineering/*.md ~/.claude/agents/

# 方式二：使用安裝腳本（支援所有工具）
./scripts/install.sh --tool claude-code

# 方式三：安裝所有支援工具（互動模式）
./scripts/install.sh
```

### 指定工具安裝範例

```bash
# Cursor
./scripts/convert.sh --tool cursor
./scripts/install.sh --tool cursor

# Aider（輸出單一 CONVENTIONS.md 到當前專案目錄）
./scripts/convert.sh --tool aider
./scripts/install.sh --tool aider

# 所有工具同時轉換（使用 CPU 並行加速）
./scripts/convert.sh --parallel
./scripts/install.sh --parallel
```

> **注意**：`integrations/` 目錄下的大部分內容是 gitignored 的生成檔案。新 clone 後只有 `README.md` 存在，需先執行 `convert.sh` 才能安裝到大多數工具。

---

## 3. 新增 Agent 的完整流程

### Step 1：選擇 category 目錄

```
academic/       design/         engineering/    finance/
game-development/  marketing/   paid-media/     product/
project-management/ sales/      spatial-computing/ specialized/
strategy/       support/        testing/
```

若找不到合適的分類，放入 `specialized/`；若要新增全新分類，須同步更新三個腳本中的 `AGENT_DIRS` 陣列（見第 5 節）。

### Step 2：建立 `.md` 檔案（frontmatter 模板）

檔名規則：`{category}-{role-slug}.md`，例如 `engineering-rust-developer.md`。

```markdown
---
name: Rust Developer
description: Systems-level Rust developer specializing in memory-safe, high-performance code
color: orange
emoji: 🦀
vibe: Fearless about ownership, obsessive about zero-cost abstractions.
services:                        # 可選，只在需要外部服務時填寫
  - name: crates.io
    url: https://crates.io
    tier: free
---
```

**必填欄位**：`name`、`description`、`color`（顏色名稱或 hex，如 `"#FF6B35"`）

### Step 3：撰寫 agent body（必要 sections）

```markdown
# Rust Developer

## 🧠 Your Identity & Memory
- **Role**: 描述核心角色定位（一句話）
- **Personality**: 強烈、具體的個性描述（避免 generic 語言）

## 🎯 Your Core Mission
明確說明 agent 解決什麼問題、產出什麼價值。

## 🚨 Critical Rules
- 列出不可違反的原則（至少 3 條）

## 📋 Your Technical Deliverables
- 具體的輸出清單（帶有範例或格式說明）

## 🔄 Your Workflow Process
1. 步驟一
2. 步驟二

## 🎯 Your Success Metrics
- 量化指標（例如：「PR review 完成時間 < 30 分鐘」）
```

### Step 4：本地 lint 驗證

```bash
# 驗證單一檔案
./scripts/lint-agents.sh engineering/engineering-rust-developer.md

# 驗證整個目錄
./scripts/lint-agents.sh
```

看到 `PASS` 即可繼續；`WARN` 是建議修改但不阻擋；`ERROR` 必須修正才能通過 CI。

### Step 5：轉換測試

```bash
# 確認能成功轉換為各工具格式
./scripts/convert.sh --tool cursor
./scripts/convert.sh --tool aider
```

### Step 6：提交 PR

```bash
git checkout -b add/engineering-rust-developer
git add engineering/engineering-rust-developer.md
git commit -m "feat(engineering): add Rust Developer agent"
git push origin add/engineering-rust-developer
# 開 PR → CI 自動執行 lint-agents.yml
```

---

## 4. Agent 設計最佳實踐

### 強烈個性（避免 generic 語言）

**不好的寫法：**
```
Personality: Helpful and knowledgeable software developer.
```

**好的寫法：**
```
Personality: Pedantic about compiler warnings, physically uncomfortable
when code compiles with -O0, treats every heap allocation as a moral failing.
```

### 具體的 deliverables（附程式碼範例）

```markdown
## 📋 Your Technical Deliverables

- **Code Review Reports**：每個 issue 附上具體的程式碼片段與修正建議
  ```rust
  // ❌ 原始碼問題
  let data = vec.clone();  // 不必要的 clone
  // ✅ 建議修正
  let data = &vec;         // 借用即可
  ```
- **Performance Benchmarks**：使用 `criterion` 基準測試，附上 ns/op 數據
- **Safety Audit Checklist**：逐行標記 unsafe block 的必要性與風險
```

### 可量化的成功指標

```markdown
## 🎯 Your Success Metrics

- Code review 回應時間：收到 PR 後 < 30 分鐘完成初審
- 零 `clippy::pedantic` 警告通過率：100%
- 文件覆蓋率（public API）：> 95%
- 每次迭代至少找出 1 個效能改善點
```

---

## 5. 新增工具支援的流程

新增一個新的 AI 工具（例如 `new-tool`）需要同步修改三個地方：

### Step 1：`scripts/convert.sh` — 新增轉換函式

在 `ALL_TOOLS` 陣列中加入 `new-tool`，並新增對應的 `convert_new_tool()` 函式：

```bash
convert_new_tool() {
  local out_dir="$CONVERT_OUT/new-tool"
  mkdir -p "$out_dir"
  for dir in "${AGENT_DIRS[@]}"; do
    for f in "$REPO_ROOT/$dir"/*.md; do
      [[ -f "$f" ]] || continue
      # 解析 frontmatter、轉換格式、寫入 $out_dir
    done
  done
}
```

### Step 2：`scripts/install.sh` — 新增偵測與安裝函式

```bash
detect_new_tool() {
  # 回傳工具的設定目錄路徑
  echo "$HOME/.new-tool/agents"
}

install_new_tool() {
  local target_dir
  target_dir=$(detect_new_tool)
  mkdir -p "$target_dir"
  cp -r "$INTEGRATIONS_DIR/new-tool/." "$target_dir/"
  echo "Installed to $target_dir"
}
```

同時在 `ALL_TOOLS` 陣列中加入 `new-tool`。

### Step 3：新增 README

建立 `integrations/new-tool/README.md`，說明：
- 安裝步驟
- 工具的 agent 載入方式
- 已知限制

> **注意**：`AGENT_DIRS` 在 `convert.sh`、`install.sh`、`lint-agents.sh` 三個腳本中各有一份，若新增 category 目錄，三個地方都要同步更新。

---

## 6. 常見問題與 Debugging

### lint ERROR vs WARN 的差別

| 等級 | 意義 | CI 行為 |
|------|------|---------|
| `ERROR` | frontmatter 缺少必填欄位（`name`、`description`、`color`）或格式嚴重錯誤 | 阻擋 PR merge |
| `WARN` | 缺少建議的 sections（`Identity`、`Core Mission`、`Critical Rules`） | 不阻擋，但建議修正 |
| `PASS` | 所有檢查通過 | CI 成功 |

### `integrations/` 目錄為空怎麼辦

這是正常的。`.gitignore` 排除了所有 `convert.sh` 生成的檔案，只保留 `README.md`。

```bash
# 解法：先執行 convert.sh 生成整合格式
./scripts/convert.sh --tool cursor   # 只生成 Cursor 格式
./scripts/convert.sh                 # 生成所有格式
```

### convert.sh 和 install.sh 的正確執行順序

**必須先 convert，再 install：**

```bash
# 正確順序
./scripts/convert.sh --tool windsurf
./scripts/install.sh --tool windsurf

# 錯誤：直接 install 而沒有先 convert（integrations/ 是空的）
./scripts/install.sh --tool windsurf  # ❌ 找不到要安裝的檔案
```

唯一例外是 Claude Code 和 GitHub Copilot — 它們使用原生 `.md` 格式，`install.sh` 會直接從 repo 根目錄複製，無需先 convert。

### 並行模式的使用場景

```bash
# 適合：首次安裝所有工具（加速約 2-4x）
./scripts/convert.sh --parallel --jobs 8
./scripts/install.sh --parallel

# 不適合：CI 環境或單一工具安裝（並行開銷不划算）
./scripts/convert.sh --tool claude-code   # 單工具用序列即可
```

並行模式在轉換 11 個工具格式 × 144 個 agents 時效果最顯著。

---

## 7. Contribution Workflow

```mermaid
flowchart TD
    A([開始]) --> B[Fork repo]
    B --> C[建立 feature branch\nadd/{category}-{agent-name}]
    C --> D[建立 agent .md 檔案\n含 frontmatter + body]
    D --> E[本地執行 lint-agents.sh]
    E --> F{lint 結果}
    F -->|ERROR| G[修正 frontmatter\n必填欄位]
    G --> E
    F -->|WARN| H{是否修正\n建議 sections?}
    H -->|是| D
    H -->|否，可接受| I
    F -->|PASS| I[git commit & push]
    I --> J[開 Pull Request\n至 main branch]
    J --> K[GitHub Actions\nlint-agents.yml 自動執行]
    K --> L{CI 結果}
    L -->|失敗| M[根據 CI log 修正]
    M --> I
    L -->|通過| N[等待 maintainer review]
    N --> O{Review 結果}
    O -->|需要修改| P[根據 feedback 更新]
    P --> I
    O -->|核准| Q([Merge to main 🎉])
```

---

## 8. NEXUS 框架快速上手

**NEXUS**（Network of EXperts, Unified in Strategy）是 `strategy/` 目錄中定義的多 agent 協作框架，解決單一 agent 協作時常見的問題：衝突的架構決策、重複工作、handoff 邊界的品質落差。

### 三種執行模式

| 模式 | 適用情境 | 涉及 agents | 預估時程 |
|------|----------|-------------|----------|
| **NEXUS-Full** | 從零建立完整產品 | 全部 | 12-24 週 |
| **NEXUS-Sprint** | 功能開發或 MVP | 15-25 個 | 2-6 週 |
| **NEXUS-Micro** | 單一任務（bug fix、audit） | 5-10 個 | 1-5 天 |

### 使用方式

1. **安裝 strategy agents：**
   ```bash
   cp strategy/*.md ~/.claude/agents/
   ```

2. **啟動 NEXUS-Micro（最快上手）：**
   ```
   Activate Agents Orchestrator in NEXUS-Micro mode.

   Task: [描述你的任務，例如：review this API for security issues]
   Agents needed: [Security Auditor, Backend Architect, Code Reviewer]
   ```

3. **參考完整文件：**
   - `strategy/nexus-strategy.md` — 完整操作手冊（含 7 個執行階段、品質關卡、handoff 協議）
   - `strategy/QUICKSTART.md` — 5 分鐘入門（含可直接複製的啟動 prompt）
   - `examples/nexus-spatial-discovery.md` — 8 個 agents 同時協作的完整範例

### NEXUS 關鍵原則

- **Pipeline Integrity**：每個階段必須通過品質關卡才能推進，不跳關
- **Evidence Required**：所有評估必須附有具體證據，不接受主觀聲明
- **最多 3 次重試**：超過後自動升級（escalation），避免無限循環

---

*最後更新：2026-04-30 | 本指南由 Claude Code Agent 自動產出*
