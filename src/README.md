# Agent Team Dashboard

> Visual dashboard for AI agent teams. Pipeline + Operation Log + Process Monitor — see your whole project at a glance.

[📖 中文版 / Chinese Version →](README.zh.md)

---

## What is this?

**Running a multi-agent project?** You have Workers (Scout, Designer, Builder, Tester, Seller) writing files, committing code, running tests. How do you know what's happening right now? The Dashboard gives you a live view: which stage the project is in, what each Worker just delivered, and whether your Claude Code agents are running.

---

## Quick Start

### Path A: Just try it (zero config)

```bash
cd your-project
npx agent-team-dashboard
```

Then open **http://localhost:3456** in your browser.

**You'll see** three panels:
- **Pipeline** — 5-stage progress (Scout → Designer → Builder → Tester → Seller), detected from your `briefs/` and `src/` files
- **Operation Log** — semantic git commits timeline, grouped by Worker tag
- **Process Monitor** — live Claude Code agent status cards

No config, no API keys, no dependencies beyond Node.js ≥ 18 and Git.

### Path B: My project uses Company OS Worker templates

If your project follows the Company OS structure (`briefs/scout→designer.md`, `briefs/designer→builder.md`, etc.), the Pipeline panel automatically detects:
- Which stages are done (file exists + populated)
- Which stage is in progress (files being modified)
- Which stages are still pending

The Operation Log picks up your semantic commits (`[scout]`, `[builder]`, `[tester]`, etc.) and displays them as a timeline.

### Path C: I want to see Claude Code agent activity

The Process Monitor watches `~/.claude/sessions/` and shows live process cards:
- **Agent ID** (session directory name)
- **Status** — idle, busy, or waiting for decision
- **Project** the agent is working on
- **Idle time**

No Claude Code configuration needed — it reads the sessions directory automatically.

---

## Setup Auto-Commit Hooks (recommended)

The Dashboard shines when your Workers use semantic git commits. Set up auto-commit hooks so every file change is tracked:

```bash
mkdir -p .claude
cat > .claude/settings.json << 'EOF'
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "cd ${CLAUDE_PROJECT_DIR} && git add -A && git commit -m \"[auto] ${CLAUDE_TOOL_NAME} ${CLAUDE_FILE_PATH}\""
          }
        ]
      }
    ]
  }
}
EOF
```

Or copy from Company OS template: `cp agents/templates/settings.json .claude/settings.json`

---

## Semantic Commit Tags

When Workers make important deliveries, they add a tag. The Dashboard groups commits by these tags:

| Tag | Worker | Example |
|-----|--------|---------|
| `[scout]` | Scout | `[scout] Competitor analysis complete` |
| `[designer]` | Designer | `[designer] Architecture design delivered` |
| `[builder]` | Builder | `[builder] Pipeline panel complete` |
| `[tester]` | Tester | `[tester] 38/38 tests pass` |
| `[seller]` | Seller | `[seller] Package published` |
| `[auto]` | System | `[auto] Write briefs/scout→designer.md` |

---

## Options

```bash
npx agent-team-dashboard              # Default: port 3456
npx agent-team-dashboard --port 8080  # Custom port
PORT=8080 npx agent-team-dashboard    # Via environment variable
```

---

## Troubleshooting

<details>
<summary><b>Dashboard starts but Operation Log is empty</b></summary>

The Operation Log reads from `git log`. Make sure:
1. Your project has Git initialized (`git init` if not)
2. You have at least one commit with a `[tag]` prefix
3. Run `git log --oneline` to check

If the project has no Git, the Dashboard auto-initializes it on first run.
</details>

<details>
<summary><b>"detected dubious ownership" error (Windows)</b></summary>

Windows/WSL users may see this git error. It's a security feature — the Dashboard handles it gracefully (shows an error card but keeps running). To fix permanently:

```bash
git config --global --add safe.directory 'E:/我的公司/projects/your-project'
```

Or trust all directories:
```bash
git config --global --add safe.directory '*'
```
</details>

<details>
<summary><b>Pipeline shows all stages as "pending"</b></summary>

The Pipeline checks for specific files:
- Scout → `briefs/scout→designer.md`
- Designer → `briefs/designer→builder.md`
- Builder → `src/` (non-empty directory)
- Tester → `src/tests/e2e.test.js`
- Seller → `seller/readme.md`

If these files don't exist, stages show as pending. Create the files and the Pipeline updates in real time (chokidar watches for changes).
</details>

<details>
<summary><b>Process Monitor shows "0 agents"</b></summary>

Process Monitor reads from `~/.claude/sessions/`. If you don't have Claude Code running, you'll see 0 agents. This is normal — the panel still works, it just has nothing to display yet.

Start a Claude Code session and the monitor updates within seconds.
</details>

<details>
<summary><b>Port 3456 is already in use</b></summary>

Use a different port: `npx agent-team-dashboard --port 3457`
</details>

---

## Requirements

| Dependency | Version | Notes |
|-----------|---------|-------|
| Node.js | ≥ 18 | Runtime |
| Git | Any | Operation Log needs it; auto-init if missing |

Claude Code is optional — only needed for Process Monitor.

---

## Tech Stack

Pure Node.js + vanilla HTML/CSS/JS. One runtime dependency: `chokidar` (file watcher). SSE for real-time updates. Zero build steps.

---

## Links

- [GitHub](https://github.com/JIANGXIADADAO/agent-team-dashboard)
- [npm](https://www.npmjs.com/package/agent-team-dashboard)
