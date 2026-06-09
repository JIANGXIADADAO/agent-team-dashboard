# Agent Team Dashboard

> Visual dashboard for AI agent teams. Pipeline + Operation Log + Process Monitor — see your whole project at a glance.

[📖 中文版](src/README.zh.md) · [npm](https://www.npmjs.com/package/agent-team-dashboard) · [Install & Docs →](src/README.md)

---

## Quick Start

```bash
npx agent-team-dashboard
```

That's it. Opens at `http://localhost:3456`. Zero config, zero API keys.

## What you get

| Panel | Data Source | What it shows |
|-------|------------|---------------|
| **Pipeline** | `briefs/` + `src/` (chokidar) | 5-stage progress: Scout → Designer → Builder → Tester → Seller |
| **Operation Log** | `git log` (5s polling) | Semantic commits timeline, grouped by Worker tag |
| **Process Monitor** | `~/.claude/sessions/` | Live Claude Code agent status cards |

## Project Structure

This is a [Company OS](https://github.com/JIANGXIADADAO/Company-OS) project. The npm package lives in `src/`.

```
agent-team-dashboard/
├── README.md              ← you are here
├── briefs/                ← design briefs (scout→designer, designer→builder, builder→tester)
├── scout/                 ← competitor analysis
├── designer/              ← requirements, architecture, interaction design
├── builder/               ← CLI reference
├── tester/                ← test plan, fix prompts
├── seller/                ← landing page, release notes
└── src/                   ← npm package (server + frontend)
    ├── README.md          ← full docs (EN)
    ├── README.zh.md       ← full docs (中文)
    └── package.json
```

## Links

- [npm](https://www.npmjs.com/package/agent-team-dashboard)
- [Company OS](https://github.com/JIANGXIADADAO/Company-OS) — the framework this project is built on
