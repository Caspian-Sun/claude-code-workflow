<div align="center">

**Language:** [**English**](https://github.com/Caspian-Sun/claude-code-workflow/blob/main/README.md) | [简体中文](https://github.com/Caspian-Sun/claude-code-workflow/blob/main-zh/README.md)

# Claude Code WorkFlow

<p><strong>AI 驱动的研发工作流框架</strong></p>

<p>
把「需求 → 拆解 → 实现 → 验证 → 评审 → 交付 → 发布」全链路拆成可追溯的命令、技能、子代理和规则<br />
由 AI 执行, 人监督每个关键节点 · 运行在 <a href="https://docs.claude.com/en/docs/claude-code">Claude Code</a> 之上
</p>

<p>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="License" /></a>
  <a href="https://docs.claude.com/en/docs/claude-code"><img src="https://img.shields.io/badge/Powered%20by-Claude%20Code-8A2BE2" alt="Powered by Claude Code" /></a>
  <img src="https://img.shields.io/badge/status-active-success.svg" alt="Status" />
  <img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg" alt="PRs Welcome" />
  <br />
  <a href="https://github.com/Caspian-Sun/claude-code-workflow/stargazers"><img src="https://img.shields.io/github/stars/Caspian-Sun/claude-code-workflow?style=social" alt="Stars" /></a>
  <a href="https://github.com/Caspian-Sun/claude-code-workflow/commits"><img src="https://img.shields.io/github/last-commit/Caspian-Sun/claude-code-workflow" alt="Last commit" /></a>
  <a href="https://github.com/Caspian-Sun/claude-code-workflow/issues"><img src="https://img.shields.io/github/issues/Caspian-Sun/claude-code-workflow" alt="Issues" /></a>
  <a href="https://github.com/Caspian-Sun/claude-code-workflow/releases"><img src="https://img.shields.io/github/v/release/Caspian-Sun/claude-code-workflow?include_prereleases&sort=semver" alt="Release" /></a>
</p>

<p>
  <strong><a href="docs/WORKFLOW.md">操作手册</a></strong> ·
  <strong><a href="docs/ADAPTING.md">跨工种适配</a></strong> ·
  <strong><a href="docs/DECISIONS.md">架构决策</a></strong> ·
  <strong><a href=".claude/README.md">框架机制</a></strong> ·
  <strong><a href="#怎么用">快速开始</a></strong>
</p>

</div>

---

## 目录

- [这是什么](#这是什么)
- [核心亮点](#核心亮点)
- [框架五件套](#框架五件套)
- [八步法骨架](#八步法骨架)
- [三大设计原则](#三大设计原则)
- [现实世界的验证](#现实世界的验证)
- [目录总览](#目录总览)
- [怎么用](#怎么用)
- [分支说明 & 姐妹仓库](#分支说明--姐妹仓库)
- [我该从哪里开始看](#我该从哪里开始看)
- [许可证](#许可证)

---

## 这是什么

**本仓库是框架本体, 与具体工种 (前端 / 后端 / 数据 / 移动 / DevOps / QA / 设计 / 产品 / 写作 / 科研...) 无关。** 具体实现代码放在 `workspace/` 下, 由使用者按工种替换。

AI 协作研发的两个老问题, 框架用闸门和档案直接回答:

| 问题          | 表现               | 框架怎么治                                                        |
| ------------- | ------------------ | ----------------------------------------------------------------- |
| **AI 没约束** | 瞎写、乱改、绕 bug | 硬闸门 (需求完备性 / `@rules` 追溯链 / 禁硬编码 / Hooks 静默守护) |
| **AI 没记忆** | 每次会话从零猜     | 档案沉淀 (ADR / retrospectives / tasks.json) 跨会话可读           |

> 框架**不绑技术栈, 不绑领域**。工种特化在 `.claude/rules/*.md` 和 `workspace/` 里做。

---

## 核心亮点

- **八步法 SDLC** — `/prd → /plan → /code → /test → /review → /build → /deploy → /release` 一条流水线, 辅以 `/fix` `/bug-check` `/prd-check` `/plan-check` `/start` `/meta-audit`
- **硬性闸门** — `prd-check` 拦占位符, `plan-check` 拦不完整任务, AI 无法默默跳过
- **可追溯链** — 需求锚点 → 任务 ID → 产出物 `@prd/@rules` → 验证 `it()`, 任何一环改动能反向扫下游
- **五件套协作** — 命令 (决策) + 技能 (脚本) + 子代理 (并行/独立视角) + 钩子 (静默守护) + 规则 (长期约束), 边界清晰
- **跨工种可移植** — 换 `workspace/` + 重写 `.claude/rules/` 内容, 即可用于后端 / 数据 / 移动 / 其他任意工种 (见 [ADAPTING.md](docs/ADAPTING.md))
- **档案自带上下文** — ADR 记决策, retrospectives 记健康度, tasks.json 记进度; 新 session 进来一眼看懂历史

---

## 框架五件套

| 部件                | 位置                                   | 触发方式       | 适合                |
| ------------------- | -------------------------------------- | -------------- | ------------------- |
| **命令** (commands) | [.claude/commands/](.claude/commands/) | 用户 `/<name>` | 主工作流 (纯思考)   |
| **技能包** (skills) | [.claude/skills/](.claude/skills/)     | 显式或 AI 自动 | 跑脚本拿数据        |
| **子代理** (agents) | [.claude/agents/](.claude/agents/)     | 主命令 spawn   | 并行 / 保护 context |
| **钩子** (hooks)    | [.claude/hooks/](.claude/hooks/)       | 事件自动       | 静默守护 (不阻断)   |
| **规则** (rules)    | [.claude/rules/](.claude/rules/)       | AI 自动遵守    | 长期稳定的产出约束  |

边界和添加规范详见 [.claude/README.md](.claude/README.md)。

---

## 八步法骨架

```text
 /prd       口语需求 ──→ 结构化需求 (含 [待确认])
    │
    │  人工审, /prd-check 清零占位符
    ▼
 /plan      需求 ──→ 任务清单 (含 prdRef + 业务规则)
    │
    │  /plan-check 验收
    ▼
 /code      任务清单 ──→ 产出物 (头部挂 @prd / @task / @rules)
    ▼
 /test      产出物 @rules ──→ 验证用例 (每条规则一条)
    ▼
 /review    独立视角审查 (可 spawn code-reviewer)
    ▼
 /build     产物化
    ▼
 /deploy    交付到目标环境
    ▼
 /release   聚合 changelog + 打 tag
```

每步的**抽象语义跨工种通用**, 产出物按工种替换。完整操作手册见 [docs/WORKFLOW.md](docs/WORKFLOW.md)。

---

## 三大设计原则

<table>
<tr>
<td width="33%" valign="top">

### 可追溯 (Traceable)

需求锚点 → 任务 ID → 产出物 `@prd/@rules` → 验证用例, 一条线贯穿。

任何一环改了, 顺着链路扫下游, 不漏不错位。

</td>
<td width="33%" valign="top">

### 关键节点人审

AI 做全量执行, 但需求 / 拆解 / 评审 / 交付前都**停下等人点头**。

AI 不能默默绕过闸门, `/prd-check` `/plan-check` 是硬拦截, 不是提醒。

</td>
<td width="33%" valign="top">

### 失败显式可见

不隐藏错误, 不自动绕过, 不以「通过」掩盖 bug。

红的时候按 4 类分诊 (工具 → 环境 → 预期 → 产出), **产出是最后才怀疑的**。

</td>
</tr>
</table>

---

## 现实世界的验证

这套框架不是纸上谈兵 —— 已经在 **3 个完全不同的栈、3 种部署形态、3 个不同领域**里被复用过, 框架本体一行没改。每个项目都自带一份 `methodology-report.md`, 诚实记录"哪些做对了, 哪些没解决"。

| 项目 | 领域 | 技术栈 | 状态 | 证明了什么 |
|------|------|--------|------|-----------|
| [`ai-frontend`](../../tree/ai-frontend) | Web 管理后台 | UmiJS + React + Antd + Vitest + Playwright | 参考实现 | 框架的原始诞生地 —— 闸门、`@rules`、追溯链等机制都是从这里长出来的 |
| [**Spider**](https://github.com/Caspian-Sun/spider) | 桌面端开发者工具 | Tauri 2 + Rust + React | 开发中 · 16 commits | **跨栈**: 从纯 TS 跨到 Rust + TS 双栈, PRD 锚点从 JSDoc 平滑迁移到 Rust `///` doc, IPC 引入了新的"禁硬编码"红线 |
| **Cpcash Wallet** *(私有仓库, 可联系作者访问)* | Web3 移动端钱包 | Flutter (Dart) · Android · iOS · **鸿蒙 OHOS** | **生产级 · 138 commits · 二期进行中** | **跨平台 + 跨领域**: 167 个任务 100% 完成度可追溯, 4 个 bug 全闭环修复, dev + staging release 13 天内交付 |

### 关键数据

**Cpcash Wallet** (最高保真度的案例):
- **13 天 138 commits**, 每个 commit 都挂任务 ID (`T001-T043 — 43/43 = 100%`)
- **167 个任务 100% 完成**, 横跨 5 个模块 (account / assets / home / transfer / receive)
- **4 份结构化 bug 报告** 驱动 `/fix` 全部闭环 (bug 密度 ≈ 2.4%, 对 Web3 来说极低)
- **一份 Dart 代码同时跑 Android / iOS / 鸿蒙 OHOS 三端**, 鸿蒙特有的 `MissingPluginException` 在同一分支顺手解决, 没有分裂出鸿蒙独立分支

**Spider** (元意义案例):
- 这个产品**自身**就是把任意 `claude-code-workflow` 仓库可视化成看板 —— **用方法论造的、用来可视化方法论的工具**
- React (15 个 feature 模块) + Rust 后端 (PTY / scan / watcher / IPC), 同一套 `@prd / @task / @rules` 链条横跨两个语言不变形
- 证明框架的"领域层"可以是开发者工具产品本身, 不只是业务 App

### 这两个项目也暴露了什么 (诚实记录)

- **Spider 16 commits 之后节奏放缓** —— 反映出八步法的前置 PRD 成本对"边设计边实现的探索性工具"偏高, 是一个值得写进"不适用场景"的真实边界案例
- **Cpcash 的测试偏 UI bug 报告, 而非 widget 测试** —— 方法论两类都支持, 但没有对 Flutter 项目强制要求平衡
- **Token 经济性目前还没量化** —— 在推荐给"成本敏感团队"之前需要补这块数据

> 完整复盘报告: [Spider 方法论复盘](https://github.com/Caspian-Sun/spider/blob/main/docs/methodology-report.md) · Cpcash 方法论复盘 (私有仓库, 可联系作者获取)

---

## 目录总览

```text
claude-code-work/
├── README.md                 ← 你在这 (框架本体介绍)
├── CLAUDE.md                 ← 项目规则 (Claude Code 启动时自动加载)
├── .claude/                  ← 框架本体 (领域无关的机制)
│   ├── commands/             ← 八步法命令 + 辅助命令
│   ├── skills/               ← 扩展技能包
│   ├── agents/               ← 专职子代理
│   ├── hooks/                ← 事件钩子
│   └── rules/                ← 产出约束 (换工种时重写内容, 保留结构)
├── docs/                     ← AI 工作流产物 + 历史档案
│   ├── WORKFLOW.md           ← 八步法操作手册
│   ├── ADAPTING.md           ← 跨工种适配清单 (fork 本框架时必读)
│   ├── DECISIONS.md          ← 架构决策记录 (ADR)
│   ├── prds/                 ← /prd 生成的需求文档
│   ├── tasks/                ← /plan 生成的任务清单 (JSON)
│   ├── bug-reports/          ← /fix 的输入
│   └── retrospectives/       ← /meta-audit 产出的只读快照
└── workspace/                ← 实际业务工程 (可替换为任何工种)
```

- **框架本体** = `.claude/` + `docs/` + `CLAUDE.md` + `README.md`
- **工种特化** = `workspace/` + `.claude/rules/*.md` 的具体内容

---

## 怎么用

### 前置要求

- [Claude Code CLI](https://docs.claude.com/en/docs/claude-code) 已安装并登录

### 三步起步

```bash
# 1. Fork 本仓库, 改名为你自己的
git clone https://github.com/Caspian-Sun/claude-code-workflow.git ai-<工种>-automation
cd ai-<工种>-automation

# 2. 按 ADAPTING.md 替换工种特化层
#    - 替换 workspace/ 为你工种的项目骨架
#    - 重写 .claude/rules/*.md 里的规则内容
#    - 更新 CLAUDE.md 的入职培训

# 3. 打开 Claude Code, 跑第一个需求
claude
> /start                       # 首次必做, AI 通读项目
> /prd <你的需求描述>          # 生成需求草稿
```

框架本体**无需额外安装**, 有 Claude Code CLI 就能跑。`workspace/` 的依赖由具体工种决定 (前端 `pnpm install`, 后端可能是 `mvn / go mod / pip install`, 等)。

---

## 分支说明 & 姐妹仓库

### 本仓库内的分支

| 分支          | 定位                                                      |
| ------------- | --------------------------------------------------------- |
| `main`        | 框架本体介绍 (与工种无关, 你在这)                         |
| `ai-frontend` | 前端工种实现 (UmiJS + React + antd + Vitest + Playwright) |
| `feature`     | 开发集成分支                                              |
| `Harness`     | 框架本体迭代分支                                          |

### 姐妹仓库 (用本方法论造出来的)

| 仓库 | 是什么 | 为什么值得看 |
|------|--------|-------------|
| [**Spider**](https://github.com/Caspian-Sun/spider) | Tauri 2 桌面工具, 把任意 `claude-code-workflow` 仓库可视化成看板 | 证明框架在 **Rust + React 双栈**和**开发者工具产品**这种形态里都能跑 |
| **Cpcash Wallet** *(私有仓库)* | Flutter Web3 钱包 (Android · iOS · 鸿蒙) | 证明框架在 **Flutter / Dart**、**三端移动 OS**、**Web3 高门槛领域**、**生产级规模**(138 commits / 167 任务 100% 完成)都能撑住 |

想看具体工种的完整示例, 切到 `ai-frontend` 分支或翻上面的姐妹仓库。未来会陆续补 `ai-backend` / `ai-data` 等分支。

---

## 我该从哪里开始看

| 我是...                               | 第一个打开                                                                |
| ------------------------------------- | ------------------------------------------------------------------------- |
| **第一次接手框架**                    | [docs/WORKFLOW.md](docs/WORKFLOW.md) — 八步法操作手册                     |
| 想把框架迁移到某个工种                | [docs/ADAPTING.md](docs/ADAPTING.md) — 跨工种适配清单                     |
| 要改框架机制 (加命令 / skill / agent) | [.claude/README.md](.claude/README.md) — 五类部件的边界                   |
| 查框架怎么演变的                      | [docs/DECISIONS.md](docs/DECISIONS.md) — 架构决策记录 (ADR)               |
| 查历次健康度扫描                      | [docs/retrospectives/](docs/retrospectives/) — `/meta-audit` 只读观察报告 |
| 启用 GitHub 自动化                    | [.github/SETUP.md](.github/SETUP.md)                                      |

---

## 贡献

欢迎 Issue 和 PR。提交前请:

1. 跑 `/meta-audit` 看有没有引入健康度问题
2. 如果改了框架机制, 在 [docs/DECISIONS.md](docs/DECISIONS.md) 加一条 ADR 说明背景

---

## 许可证

[MIT](LICENSE) © 2026 suntaoTom
