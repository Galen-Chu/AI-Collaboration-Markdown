# 📄 AI-Collaboration-Markdown

> **Project Documentation Spec** — Twenty markdown files every project ships,
> with clear boundaries, filled examples, and cross-reference rules.

---

## 🧭 Core Philosophy · 核心理念

Each file answers **one question**. Mixing them is the usual failure mode —
a README bloated with architecture, or a spec that's secretly a changelog.

每份文件只回答**一個問題**。混合是多數專案的常見失敗模式——README 塞滿架構、
spec 混入 changelog。這份規格定義了 20 個標準文件，讓邊界清晰、職責分明。

---

## 📋 The Twenty Files · 二十份文件

| # | File | Question It Answers | Audience | Written When |
|---|------|---------------------|----------|-------------|
| 1 | `README` | What is this, how do I run it | Anyone landing on the repo | Project start |
| 2 | `INDEX` | Where's everything, what's stale | Anyone navigating | Project start, kept live |
| 3 | `CLAUDE` | Rules for AI agents in this repo | AI agents, contributors | Project start |
| 4 | `SYSTEM` | What's shared across all my projects | You, across projects | Once, kept current |
| 5 | `DESIGN` | Why this approach, not another | Team, before building | Before building |
| 6 | `ROADMAP` | What's next, and when | Team, stakeholders | After DESIGN, kept live |
| 7 | `PLAN` | Who's doing what, this milestone | Team, this sprint | Start of each milestone |
| 8 | `SPECIFICATION` | What it does, precisely | Implementers | Before building, on change |
| 9 | `ARCHITECTURE` | What's in the stack | Engineers, onboarding | Before building, on change |
| 10 | `TECH` | Which versions, flat inventory | Engineers, onboarding | Before building, on change |
| 11 | `IMPLEMENTATION` | How it was actually built | Future maintainers | During / after building |
| 12 | `SKILL` | How to do one recurring task | Implementer, AI agents | First time task recurs |
| 13 | `CRITERIA` | What "done" means, per feature | QA, stakeholders | During planning |
| 14 | `ACCEPTANCE` | Did it actually pass | QA, release manager | QA / release |
| 15 | `ANALYSIS` | What did we learn | Team, future planning | After release / incident |
| 16 | `LOG` | What happened, dated | Future debugger | Continuous |
| 17 | `TEST` | How to test it | QA, CI, engineers | Before building, on change |
| 18 | `TROUBLESHOOTING` | How to fix it when it breaks | On-call, future debugger | When issues found |
| 19 | `OPERATION` | How to run this in prod | On-call, ops | Before first deploy |
| 20 | `VERSION` | What shipped, per release | Anyone landing on repo | Each release |

---

## 🔑 Key Boundaries · 關鍵邊界

The hardest part is knowing which file gets which content:

| Pair | Boundary | Example |
|------|----------|---------|
| `CRITERIA` vs `ACCEPTANCE` | Spec vs. record | CRITERIA is written up front; ACCEPTANCE is filled in later with pass/fail |
| `TEST` vs `TROUBLESHOOTING` | Proactive vs. reactive | TEST verifies before something breaks; TROUBLESHOOTING fixes it after |
| `ARCHITECTURE` vs `IMPLEMENTATION` | Intended vs. as-built | ARCHITECTURE stays clean; IMPLEMENTATION records deviations and gotchas |
| `SYSTEM` vs `ARCHITECTURE` | Cross-project vs. this-project | SYSTEM is what keeps several repos consistent |
| `DESIGN` vs `ROADMAP` | How vs. what's next | DESIGN decides how scoped work gets built; ROADMAP decides what gets scoped |
| `ROADMAP` vs `PLAN` | Strategic vs. tactical | ROADMAP spans milestones (Now/Next/Later); PLAN is one milestone's task list |
| `ANALYSIS` vs `LOG` | Interpretation vs. fact | ANALYSIS interprets what happened; LOG records what happened |

---

## 🚀 Quick Start · 快速開始

### Use the templates

```bash
git clone https://github.com/Galen-Chu/AI-Collaboration-Markdown.git
cd AI-Collaboration-Markdown

# Copy the doc set into your new project
cp -r docs/templates/ /path/to/your/project/docs/
# Fill in each file as you go
```

### Reference examples

The `docs/examples/` directory contains filled examples using a generic
service called **Atlas** — a cron scheduler — carried through all 20 files,
so you can see how they cross-reference each other.

---

## 📁 Project Structure · 專案結構

```
AI-Collaboration-Markdown/
├── docs/
│   ├── templates/          # Blank templates (20 files)
│   ├── examples/          # Filled examples ("Atlas" service)
│   └── guides/            # How-to guides for each file type
├── README.md
├── LICENSE
└── Markdown Doc Spec (standalone).html  # Original spec (interactive)
```

---

## 🔄 Integration · 整合

This spec is designed to work with:

| Project | Integration Point |
|---------|------------------|
| [AI-Agent-Skill](https://github.com/Galen-Chu/AI-Agent-Skill) | `CLAUDE` file replaces basic `.claude/` config |
| [AI-Eval-Rubric](https://github.com/Galen-Chu/AI-Eval-Rubric) | `CRITERIA` maps to rubric definitions |
| [AI-Pipeline-Hook](https://github.com/Galen-Chu/AI-Pipeline-Hook) | `LOG` and `TEST` integrate with pipeline execution |
| [Obsidian_Library](https://github.com/Galen-Chu/Obsidian_Library) | Templates compatible with Obsidian vault format |

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 👤 Author

**Galen Chu**

- GitHub: [@Galen-Chu](https://github.com/Galen-Chu)
- LinkedIn: [Galen Chu](https://www.linkedin.com/in/galen-chu-203590b5/)
