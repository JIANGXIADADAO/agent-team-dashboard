# Agent Team Dashboard

> AI Agent 团队可视化面板。Pipeline + 操作日志 + 进程监控——一个页面看清项目全貌。

[📖 English Version →](README.md)

---

## 这是什么？

**在跑多 Agent 项目？** 你有 Worker（Scout、Designer、Builder、Tester、Seller）在写文件、commit 代码、跑测试。怎么知道现在发生了什么？Dashboard 给你一个实时视图：项目在哪个阶段、每个 Worker 刚交付了什么、Claude Code Agent 是否在运行。

---

## 快速开始

### 路径 A：就想试试（零配置）

```bash
cd 你的项目目录
npx agent-team-dashboard
```

然后在浏览器打开 **http://localhost:3456**。

**你会看到** 三个面板：
- **Pipeline（流程）** — 5 阶段进度（Scout → Designer → Builder → Tester → Seller），根据 `briefs/` 和 `src/` 文件自动检测
- **Operation Log（操作日志）** — 语义 git commit 时间线，按 Worker 标签分组
- **Process Monitor（进程监控）** — Claude Code Agent 实时状态卡片

零配置、零 API key、只需 Node.js ≥ 18 和 Git。

### 路径 B：我的项目用了 Company OS Worker 模板

如果你的项目遵循 Company OS 结构（`briefs/scout→designer.md`、`briefs/designer→builder.md` 等），Pipeline 面板会自动检测：
- 哪些阶段已完成（文件存在且有内容）
- 哪个阶段进行中（文件正在被修改）
- 哪些阶段待开始

Operation Log 会自动抓取语义 commit（`[scout]`、`[builder]`、`[tester]` 等），展示为交付时间线。

### 路径 C：我想看 Claude Code Agent 活动

Process Monitor 监听 `~/.claude/sessions/`，实时显示进程卡片：
- **Agent ID**（session 目录名）
- **状态** — idle（空闲）、busy（工作中）、decision（等待决策）
- **所在项目**
- **空闲时长**

无需配置 Claude Code——自动读取 sessions 目录。

---

## 设置自动存盘 Hook（推荐）

Dashboard 在 Worker 使用语义 git commit 时最好用。设置自动存盘让每次文件改动都被追踪：

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

或从 Company OS 模板直接复制：`cp agents/templates/settings.json .claude/settings.json`

---

## 语义 Commit 标签

Worker 在关键交付点打标签 commit。Dashboard 按标签分类展示：

| 标签 | Worker | 示例 |
|------|--------|------|
| `[scout]` | Scout | `[scout] 竞品分析完成` |
| `[designer]` | Designer | `[designer] 架构设计交付` |
| `[builder]` | Builder | `[builder] Pipeline 面板完成` |
| `[tester]` | Tester | `[tester] 38/38 全部通过` |
| `[seller]` | Seller | `[seller] 产品已发布` |
| `[auto]` | 系统 | `[auto] Write briefs/scout→designer.md` |

---

## 参数

```bash
npx agent-team-dashboard              # 默认：端口 3456
npx agent-team-dashboard --port 8080  # 自定义端口
PORT=8080 npx agent-team-dashboard    # 环境变量方式
```

---

## 常见问题

<details>
<summary><b>Dashboard 启动了但 Operation Log 是空的</b></summary>

Operation Log 读的是 `git log`。检查：
1. 项目有没有 Git（没有的话 Dashboard 首次启动会自动 `git init`）
2. 有没有至少一条带 `[tag]` 前缀的 commit
3. 跑 `git log --oneline` 看看

</details>

<details>
<summary><b>"detected dubious ownership" 报错（Windows）</b></summary>

Windows/WSL 用户可能遇到这个 git 安全提示。Dashboard 会优雅处理（显示错误卡片但不崩溃）。永久修复：

```bash
git config --global --add safe.directory 'E:/我的公司/projects/你的项目'
```

或者信任所有目录：
```bash
git config --global --add safe.directory '*'
```
</details>

<details>
<summary><b>Pipeline 所有阶段都显示 "pending"</b></summary>

Pipeline 检查的是特定文件：
- Scout → `briefs/scout→designer.md`
- Designer → `briefs/designer→builder.md`
- Builder → `src/`（非空目录）
- Tester → `src/tests/e2e.test.js`
- Seller → `seller/readme.md`

如果这些文件不存在，阶段就显示 pending。创建文件后 Pipeline 实时更新（chokidar 监听文件变化）。
</details>

<details>
<summary><b>Process Monitor 显示 "0 agents"</b></summary>

Process Monitor 读的是 `~/.claude/sessions/`。如果你没有在跑 Claude Code，就是 0 agents。这是正常的——面板正常工作，只是暂无数据。

启动一个 Claude Code session，几秒内就会更新。
</details>

<details>
<summary><b>端口 3456 被占用了</b></summary>

换一个端口：`npx agent-team-dashboard --port 3457`
</details>

<details>
<summary><b>我是新手，对命令行不熟……</b></summary>

没关系。你只需要三步：
1. 打开终端
2. `cd` 到你的项目目录
3. 跑 `npx agent-team-dashboard`
4. 浏览器打开 `http://localhost:3456`

不需要 git tag、不需要分支、不需要 API key。如果项目还没有 Git，Dashboard 会自动帮你初始化。
</details>

---

## 环境要求

| 依赖 | 版本 | 说明 |
|------|------|------|
| Node.js | ≥ 18 | 运行时 |
| Git | 任意版本 | Operation Log 需要；没有的话自动初始化 |

Claude Code 可选——只有 Process Monitor 需要。

---

## 技术栈

纯 Node.js + 原生 HTML/CSS/JS。运行时只有一个依赖：`chokidar`（文件监听）。SSE 实时推送。零构建步骤。

---

## 链接

- [GitHub](https://github.com/JIANGXIADADAO/agent-team-dashboard)
- [npm](https://www.npmjs.com/package/agent-team-dashboard)
