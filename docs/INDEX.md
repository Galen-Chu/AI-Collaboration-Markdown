# 📚 Documentation Index

> Where's everything · 一覽所有文件

---

## 📖 Technical Guides · 技術指南

深入探討 AI 協作的核心技術，每份 20-40 分鐘。

| # | Document | Focus | Updated |
|---|----------|-------|---------|
| 1 | [🔌 MCP Server 實作指南](./01-mcp-server-implementation.md) | TypeScript + Python 雙語實作、我的生態系 MCP 整合（Notion vs Obsidian 分工） | 2026-08 |
| 2 | [💡 Prompt Engineering 實戰技巧](./02-prompt-engineering.md) | Agent Description 三段式公式、Claude Code 特有模式（CLAUDE.md 設計） | 2026-08 |
| 3 | [🛠️ Tool Calling 應用場景](./03-tool-calling-use-cases.md) | G-Code 生態系四個實戰案例（CI/CD、四層委派、評估閘門、Obsidian MCP） | 2026-08 |

---

## 📄 Documentation Spec · 文件規格

20 文件專案文件規格的模板與範例。

| Directory | Contents | Use |
|-----------|----------|-----|
| [templates/](./templates/) | 20 個空白模板 | 複製到新專案填入 |
| [examples/](./examples/) | 7 個填好的範例（Atlas 服務） | 參考如何交叉引用 |

### The 20 Files

```
README → INDEX → CLAUDE → SYSTEM → DESIGN → ROADMAP → PLAN
→ SPECIFICATION → ARCHITECTURE → TECH → IMPLEMENTATION → SKILL
→ CRITERIA → ACCEPTANCE → ANALYSIS → LOG → TEST
→ TROUBLESHOOTING → OPERATION → VERSION
```

完整定義見[根目錄 README](../README.md) 的對照表。

---

## 🔗 Ecosystem · 生態系

以下主題已有獨立 repo 深入探討：

| Topic | Repository | Covers |
|-------|-----------|--------|
| Agent/Skill 架構 | [AI-Agent-Skill](https://github.com/Galen-Chu/AI-Agent-Skill) | 四層架構、14 Agent + 10 Skill、MCP 整合 |
| CI/CD 自動化 | [program-g-code](https://github.com/Galen-Chu/program-g-code) | 工具包腳本、閉環自測試 |
| 排程 Pipeline | [AI-Pipeline-Hook](https://github.com/Galen-Chu/AI-Pipeline-Hook) | Hook → Pipeline → Skills → Output |
| 評估驗收 | [AI-Eval-Rubric](https://github.com/Galen-Chu/AI-Eval-Rubric) | 5 種評估規格、評估執行器 |
| 知識庫 | [Obsidian_Library](https://github.com/Galen-Chu/Obsidian_Library) | LLM 編譯器架構的知識管理 |

---

## 📋 Reading Path · 閱讀路徑

```
10 分鐘速覽
├── MCP 是什麼？ → 01 的「什麼是 MCP Server？」
├── Prompt 基礎 → 02 的「核心原則」
└── Tool Calling 實例 → 03 的「實作案例」

深入實作
├── MCP 從零開始 → 01 的「TypeScript 實作」或「Python 實作」
├── Agent 架構設計 → 02 的「Agent Description 公式」
└── 生態系整合 → 03 的「我的生態系實戰案例」

匯入新專案
└── 複製 templates/ 到你的專案 → 填入 → 參考 examples/ 的交叉引用
```

---

**Back to:** [README.md](../README.md)
