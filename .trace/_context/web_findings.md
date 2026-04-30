# Stage 1 Web Search 發現

## 搜尋摘要

### 搜尋 1: 專案概覽與 GitHub 資訊

**查詢**: `msitarzewski agency-agents GitHub AI agent prompts collection 2024 2025`

**關鍵發現**:
- 專案已在短短幾天內累積 **50,000+ Stars** 和 **7,500+ Forks**
- 起源於 Reddit 討論串（r/ClaudeAI），12 小時內收到 50+ 個請求
- 社群影響力：有多個 fork 和 community translations（zh-CN）
- PR #117 提議新增 GitHub Action 讓 agent 可在 CI/CD workflow 中使用
- YUV.AI Blog 有一篇介紹文章：[Agency Agents: Transform Your IDE into a Multi-Agent AI Studio](https://yuv.ai/blog/agency-agents)

**來源**:
- [GitHub - msitarzewski/agency-agents](https://github.com/msitarzewski/agency-agents)
- [The Agency: 144 Specialized AI Agents, 50K GitHub Stars in Days](https://www.git.edu.kg/en/posts/2026-03-17-agency-agents/)

### 搜尋 2: GitHub 倉庫詳細資訊

**查詢**: `site:github.com msitarzewski agency-agents`

**關鍵發現**:
- 維護者個人頁面：https://github.com/msitarzewski
- 有 GitHub Discussions 功能啟用，用於分享成功案例
- `examples/README.md` 包含多 agent 協作範例的說明
- `design/` 目錄有專屬的瀏覽頁面

### 搜尋 3: 社群討論與病毒式傳播

**查詢**: `agency-agents Reddit Claude Code AI agents viral 50k stars discussion`

**關鍵發現**:
- Twitter/X 上有顯著的病毒式傳播：
  > "🚨BREAKING: Someone just open sourced a complete AI agency and it hit 50K GitHub stars in under two weeks." — @heygurisingh
- 定位清晰：「不是 prompt template，是 147 個跨 12 個部門的 specialized AI agents」
- 社群把這個專案定性為「將 IDE 轉變為 multi-agent AI 工作室的工具」
- YouTube 上有相關影片：「I Built an ENTIRE Fake Company With Claude Code (Sub Agent Swarm)」

**來源**:
- [Twitter/X @heygurisingh](https://x.com/heygurisingh/status/2033966864825782543)

### 搜尋 4: Claude Code Sub-Agent 生態系統

**查詢**: `"agency-agents" Claude Code sub-agents MCP multi-agent workflow tutorial`

**關鍵發現**:
- Claude Code 官方文件有 sub-agents 功能說明：https://code.claude.com/docs/en/sub-agents
- Sub-agents 預設繼承主對話的所有工具（包括 MCP 工具）
- 「Agent Teams」是一個實驗性功能，讓多個 Claude 實例可以並行工作並透過 git 協調
- Medium 有入門教學：「Claude Code and Subagents: How to Build Your First Multi-Agent Workflow」
- agency-agents 的 `.md` + YAML frontmatter 格式正好是 Claude Code 原生的 sub-agent 格式

**來源**:
- [Claude Code Sub-agents 官方文件](https://code.claude.com/docs/en/sub-agents)
- [Medium: Claude Code Multi-Agent Workflow](https://medium.com/@techofhp/claude-code-and-subagents-how-to-build-your-first-multi-agent-workflow-3cdbc5e430fa)

## 關鍵 Takeaways

1. **市場時機**: 專案在 2026 年初 Claude Code sub-agents 功能正式推出後爆紅，定義了這個生態系的早期 agent 庫標準

2. **格式標準化**: 專案的 `.md` + YAML frontmatter 格式已成為多個 AI 工具（Claude Code, GitHub Copilot）的原生格式，無需轉換即可使用

3. **競品生態**: 已有社群維護的 translation fork（zh-CN），以及衍生的 [awesome-openclaw-agents](https://github.com/mergisi/awesome-openclaw-agents)

4. **未來路線圖**: README 中提到計劃中的功能包括 interactive agent selector web tool、community agent marketplace、agent personality quiz

5. **技術本質**: 這不是傳統意義的「software project」，而是一個 **AI agent persona library** — 每個 `.md` 檔案是一個精心設計的 system prompt，透過 toolchain 部署到各 AI coding assistant

## 相關社群資源

| 資源類型 | 連結 |
|----------|------|
| GitHub 倉庫 | https://github.com/msitarzewski/agency-agents |
| GitHub Discussions | https://github.com/msitarzewski/agency-agents/discussions |
| zh-CN 翻譯（jnMetaCode） | https://github.com/jnMetaCode/agency-agents-zh |
| zh-CN 翻譯（dsclca12） | https://github.com/dsclca12/agent-teams |
| awesome-openclaw-agents | https://github.com/mergisi/awesome-openclaw-agents |
| YUV.AI 介紹文章 | https://yuv.ai/blog/agency-agents |
| Claude Code Sub-agents 文件 | https://code.claude.com/docs/en/sub-agents |
