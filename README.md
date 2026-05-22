# TPR Framework (Think / Probe / Review)

<div align="center">
  <img src="https://img.shields.io/badge/OpenClaw-Skill-blue.svg" alt="OpenClaw Skill">
  <img src="https://img.shields.io/badge/version-3.0.0-green.svg" alt="Version 3.0.0">
</div>

> **将复杂问题从模糊需求转化为可验证、可执行、可复盘的结果。**

TPR Framework 是一个开源的 AI Agent 协作方法论。它定义了一套从认知到执行的完整工作流程，适用于 OpenClaw 及其他 Multi-Agent 平台。

---

## 核心特性

- **T/P/R 认知闭环** — Think（定义问题）→ Probe（探索验证）→ Review（决策收敛）
- **三层四阶段执行** — 编排者 / 策划层 / 审查层 / 执行层，阶段化流转
- **GRV 契约化方案** — Goal / Result / Variables，强制量化验收标准
- **Battle 对抗审查** — 审查层主动挑战方案，暴露盲区
- **Best-Minds 专家思维** — 所有 Agent 以领域顶级专家身份思考
- **双轨交付** — MD 给 AI 消费，HTML 给人审阅

---

## 安装指南

### 前提条件

| 依赖 | 要求 |
|------|------|
| OpenClaw | 已安装并正常运行 |
| Git | 已安装，能访问 GitHub |
| AI 执行器（Ralph Loop 需要） | [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 或 [OpenAI Codex](https://github.com/openclaw/codex) |

### 第一步：安装 TPR Framework

```bash
# 克隆到 OpenClaw skills 目录
git clone https://github.com/evan-zhang/tpr-framework.git ~/.openclaw/skills/tpr-framework
```

验证安装：

```bash
ls ~/.openclaw/skills/tpr-framework/SKILL.md
# 应该能看到 SKILL.md 文件
```

> OpenClaw 会自动识别 `~/.openclaw/skills/` 下的 Skill 并加载。安装后无需手动配置。

### 第二步：安装 Ralph Loop（强烈推荐）

> ⚠️ **TPR + Ralph Loop 才是完整方案。** TPR 负责从需求到方案（DISCOVERY → GRV → Battle），Ralph Loop 负责从方案到交付（Implementation）。不装 Ralph Loop，Battle 通过后无法持续执行和验证——相当于缺了一条腿。

**方式一：克隆 agent-factory 仓库（推荐）**

```bash
git clone https://github.com/evan-zhang/agent-factory.git
# Ralph Loop 位于 agent-factory/projects/2605211/ralph/
# 含：SKILL.md、scripts/（ralph-loop.sh、init-state.sh）、references/
```

**方式二：单独下载关键文件**

```bash
mkdir -p ralph/{scripts,references}

curl -o ralph/SKILL.md https://raw.githubusercontent.com/evan-zhang/agent-factory/master/projects/2605211/ralph/SKILL.md
curl -o ralph/scripts/ralph-loop.sh https://raw.githubusercontent.com/evan-zhang/agent-factory/master/projects/2605211/ralph/scripts/ralph-loop.sh
curl -o ralph/scripts/init-state.sh https://raw.githubusercontent.com/evan-zhang/agent-factory/master/projects/2605211/ralph/scripts/init-state.sh
curl -o ralph/version.json https://raw.githubusercontent.com/evan-zhang/agent-factory/master/projects/2605211/ralph/version.json
chmod +x ralph/scripts/*.sh

# 参考模板（按需）
curl -o ralph/references/prompt-template-executor.md https://raw.githubusercontent.com/evan-zhang/agent-factory/master/projects/2605211/ralph/references/prompt-template-executor.md
curl -o ralph/references/prompt-template-autonomous.md https://raw.githubusercontent.com/evan-zhang/agent-factory/master/projects/2605211/ralph/references/prompt-template-autonomous.md
```

验证安装：

```bash
bash ralph/scripts/ralph-loop.sh --help
bash ralph/scripts/init-state.sh --help
```

### 更新

```bash
# 更新 TPR Framework
cd ~/.openclaw/skills/tpr-framework && git pull origin main

# 更新 Ralph Loop（方式一安装的）
cd agent-factory && git pull origin master
```

---

## 如何使用

### 不需要配置

TPR Framework 不需要任何手动配置。安装后 Agent 自动具备以下能力：

| 能力 | 实现方式 |
|------|---------|
| 模式判定 | Agent 收到任务后自动按判定矩阵决定走"TPR 思维"还是"TPR 全流程" |
| sub-agent 检测 | Agent 自动检测运行环境是否支持 spawn，不支持则降级 |
| RT 根目录 | 首次使用时 Agent 会向你建议默认路径（`{workspace}/projects`），你可以指定其他位置 |

### 两种使用模式

#### TPR 思维（任何 Agent 可用）

不需要 sub-agent。遇到复杂问题时，按 T → P → R 顺序思考：

```
T: 我们正在解决 _______________
   成功标准是 _______________

P: 已确认 _______________
   未确认 _______________

R: 结论是 _______________
   下一步 _______________
```

**适用场景**：简单问题（<5 分钟）、紧急救火、纯执行任务、有现成 SOP 的重复操作、用户说"直接做"。

#### TPR 全流程（编排型 Agent）

需要 sub-agent 能力。完整项目生命周期：

| 阶段 | 认知重心 | 产出 | 说明 |
|------|---------|------|------|
| DISCOVERY | T + P | DISCOVERY.md | 洞察报告 |
| GRV | R | GRV.md | 契约化方案 |
| Battle | P + R | BATTLE-*.md | 对抗审查 |
| Implementation | 微型 T/P/R | output/* | 执行交付（需要 Ralph Loop） |

**适用场景**：需要正式交付物、多角色审查、阶段流转的复杂项目。

### 触发方式

直接用自然语言对 Agent 说：

- "用 TPR 分析一下 XXX 需求" → 触发 TPR 思维或全流程（Agent 自动判定）
- "走 GRV / Battle / 三层架构" → 强制触发 TPR 全流程
- 任何需要结构化分析、方案制定、决策支持的复杂问题

### 阶段流转

```
需求进入
  │
  ├─ 简单？→ TPR 思维速记（T→P→R，不产出文件）
  │
  └─ 复杂？→ TPR 全流程
       │
       ├─ DISCOVERY（策划层）：理解需求，识别假设，产出洞察报告
       │
       ├─ GRV（策划层）：起草契约化方案，量化验收标准
       │
       ├─ Battle（审查层 vs 执行层）：对抗审查，暴露盲区
       │
       └─ Implementation（执行层 + Ralph Loop）：持续执行，循环验证
            │
            └─ Closure：验收确认
```

### Implementation 的三种执行模式

| 模式 | 需要 Ralph Loop | 适用场景 |
|------|----------------|---------|
| Mode A：单线程执行 | ❌ | 简单任务，步骤已知 |
| Mode B：Ralph Loop 持续执行 | ✅ | 复杂任务，多阶段，需要循环验证 |
| Mode C：其他 coding agent | ❌ | 一次性执行 |

> 安装了 Ralph Loop 的 Agent 默认使用 Mode B（持续验证执行），这是推荐方式。

---

## 三层角色

| 角色 | 职责 | T/P/R 映射 |
|------|------|-----------|
| 编排者 | 维护节奏，协调流转，不动手 | 流程管理 |
| 策划层 | 洞察需求，起草 GRV | Think → Review |
| 审查层 | 挑战假设，暴露盲点 | Probe |
| 执行层 | 制定方案，执行交付 | 微型 T/P/R |

---

## 项目目录结构

根据项目复杂度，TPR 提供三套模板：

### 极简版（1-3 天，单一成果）
```
TPR-YYYYMMDD-NNN/
├── 01-discovery/  → DISCOVERY.md
├── 02-planning/   → GRV.md
├── 04-execution/  → 交付物
└── 05-closure/    → 验收
```

### 标准版（2 周 - 1 月，多成果）
```
TPR-YYYYMMDD-NNN/
├── 01-discovery/  → DISCOVERY.md
├── 02-planning/   → GRV.md + 成果规划
├── 03-review/     → 内部评审
├── 04-execution/  → 执行记录 + 结果
└── 05-closure/    → 验收
```

### 完整版（多目标 / 高风险）
```
TPR-YYYYMMDD-NNN/
├── 01-discovery/  → 原始需求 + 洞察报告
├── 02-planning/   → GRV + 按目标/成果分层
├── 03-battle/     → 审查层 vs 执行层对抗
├── 04-execution/  → 按举措分层执行
└── 05-closure/    → 最终验收
```

---

## 验证安装

安装完成后，可以用以下测试验证 Agent 是否正确加载了 TPR：

### 测试 1：角色认知
> 问："TPR 全流程中，你会派几个角色？如果执行层做得不好，你会亲自改吗？"

✅ 应答出：策划层、审查层、执行层三个角色；明确说"不会亲自动手，打回重跑"。

### 测试 2：流程边界
> 给一个模糊需求："下周上线 VIP 门户，帮我出方案"

✅ 应先进入 DISCOVERY 阶段，用 5 Why 追问真实需求，而不是直接输出方案。

### 测试 3：项目分级
> 问："任务很简单，1 天就能搞定，需要走全流程吗？"

✅ 应识别为极简模式，跳过 Battle，直接 GRV → 执行。

---

## 文档结构

```
tpr-framework/
├── SKILL.md                          ← 入口（Agent 自动加载）
├── README.md                         ← 本文件（安装与使用指南）
├── INSTALL.md                        ← Agent 视角安装卡片
├── CHANGELOG.md                      ← 版本历史
├── CONTRIBUTING.md                   ← 贡献指南
├── references/                       ← 按需加载的协议文档
│   ├── definition.md                 ← TPR 是什么
│   ├── tpr-cognitive.md              ← T/P/R 认知方法 + Best-Minds
│   ├── tpr-execution.md              ← 执行流程
│   ├── grv-standard.md               ← GRV 契约格式标准
│   ├── battle-protocol.md            ← Battle 机制与状态机
│   ├── orchestrator-ops.md           ← 编排操作手册
│   ├── project-grading.md            ← 项目分级
│   ├── multi-agent-pattern.md        ← 多 Agent 架构模式
│   ├── output-delivery.md            ← 产出交付协议
│   ├── tpr-bridge-protocol.md        ← TPR ↔ Ralph Loop 桥接规范
│   └── templates/                    ← 项目目录模板
├── design/                           ← 设计决策记录
└── docs/examples/                    ← 使用示例
```

---

## 常见问题

| 问题 | 回答 |
|------|------|
| Agent 说"不具备 sub-agent 能力" | 当前运行环境不支持 spawn，自动降级为 TPR 思维模式 |
| RT 文件写到了哪里 | 检查 Agent 建议的 RT 根目录，默认是 `{workspace}/projects` |
| 没装 Ralph Loop 会怎样 | Implementation 阶段自动降级为 Mode A（单线程）或 Mode C（一次性执行），功能可用但缺乏持续验证。强烈建议安装 |
| 需要编辑 AGENTS.md 吗 | 不需要。所有参数都是运行时自动检测，无需手动配置 |
| 支持哪些 AI 执行器 | Claude Code、OpenAI Codex 等，只要有 spawn sub-agent 能力即可 |

---

## 反馈与贡献

- **提交 Issue**：https://github.com/evan-zhang/tpr-framework/issues
- **贡献指南**：参见 [CONTRIBUTING.md](CONTRIBUTING.md)
- **版本历史**：参见 [CHANGELOG.md](CHANGELOG.md)

---

## 许可

MIT License
