# TPR Framework (Think / Probe / Review)

<div align="center">
  <img src="https://img.shields.io/badge/OpenClaw-Skill-blue.svg" alt="OpenClaw Skill">
  <img src="https://img.shields.io/badge/version-2.2.0-green.svg" alt="Version 2.2.0">
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
- **Consensus-Divergence Mapping** — 显式映射领域共识与分歧，防止伪共识
- **双轨交付** — MD 给 AI 消费，HTML 给人审阅

---

## 快速安装

### 前置条件

- [OpenClaw](https://github.com/openclaw/openclaw) 已安装并运行
- 一个可用的 AI Agent（如 Claude Code、Codex 等）

### 安装步骤

```bash
# 1. 克隆到 OpenClaw skills 目录
cd ~/.openclaw/skills/
git clone https://github.com/evan-zhang/tpr-framework.git

# 2. 验证安装
ls ~/.openclaw/skills/tpr-framework/SKILL.md
# 应该能看到 SKILL.md 文件

# 3. 完成！Agent 会自动识别并加载这个 Skill
```

### 升级到新版本

```bash
cd ~/.openclaw/skills/tpr-framework/
git pull origin main
```

---



## 两种使用模式

### TPR 思维（任何 Agent 可用）

遇到复杂问题时，按 T → P → R 顺序思考，不需要 sub-agent：

```
T: 我们正在解决 _______________
   成功标准是 _______________

P: 已确认 _______________
   未确认 _______________

R: 结论是 _______________
   下一步 _______________
```

### TPR 全流程（编排型 Agent）

完整项目生命周期：

| 阶段 | 产出 | 说明 |
|------|------|------|
| DISCOVERY | DISCOVERY.md | 洞察报告（T+P） |
| GRV | GRV.md | 契约化方案（R） |
| Battle | BATTLE-*.md | 对抗审查（P+R） |
| Implementation | output/* | 执行交付 |

---

## 三层角色

| 角色 | 职责 | T/P/R 映射 |
|------|------|-----------|
| 编排者 | 维护节奏，协调流转 | 流程管理 |
| 策划层 | 洞察需求，起草 GRV | Think → Review |
| 审查层 | 挑战假设，暴露盲点 | Probe |
| 执行层 | 制定方案，执行交付 | 微型 T/P/R |

---

## 项目目录结构

根据项目复杂度，TPR 提供三套模板：

### 极简版（1-3天，单一成果）
```
TPR-YYYYMMDD-NNN/
├── 01-discovery/  → DISCOVERY.md
├── 02-planning/   → GRV.md
├── 04-execution/  → 交付物
└── 05-closure/    → 验收
```

### 标准版（2周-1月，多成果）
```
TPR-YYYYMMDD-NNN/
├── 01-discovery/  → DISCOVERY.md
├── 02-planning/   → GRV.md + 成果规划
├── 03-review/     → 内部评审
├── 04-execution/  → 执行记录 + 结果
└── 05-closure/    → 验收
```

### 完整版（多目标/高风险）
```
TPR-YYYYMMDD-NNN/
├── 01-discovery/  → 原始需求 + 洞察报告
├── 02-planning/   → GRV + 按目标/成果分层
├── 03-battle/     → 审查层 vs 执行层对抗
├── 04-execution/  → 按举措分层执行
└── 05-closure/    → 最终验收
```

---



## 验收测试

安装后可以用以下测试验证 Agent 是否正确加载了 TPR：

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
├── README.md                         ← 本文件
├── CHANGELOG.md                      ← 版本历史
├── CONTRIBUTING.md                   ← 贡献指南
├── references/                       ← 按需加载的协议文档
│   ├── definition.md                 ← TPR 是什么
│   ├── tpr-cognitive.md              ← T/P/R 认知方法 + Best-Minds + 共识-分歧映射
│   ├── tpr-execution.md              ← 执行流程
│   ├── grv-standard.md               ← GRV 契约格式标准
│   ├── battle-protocol.md            ← Battle 机制与状态机
│   ├── orchestrator-ops.md           ← 编排操作手册
│   ├── project-grading.md            ← 项目分级
│   ├── multi-agent-pattern.md        ← 多 Agent 架构模式
│   ├── output-delivery.md            ← 产出交付协议
│   └── templates/                    ← 项目目录模板
│       ├── template-simple.md
│       ├── template-standard.md
│       ├── template-complex.md
│       └── manifest.json
├── design/                           ← 设计决策记录
├── docs/examples/                    ← 使用示例
└── scripts/                          ← 工具脚本
```

---

## 反馈与贡献

- **提交 Issue**：https://github.com/evan-zhang/tpr-framework/issues
- **贡献指南**：参见 [CONTRIBUTING.md](CONTRIBUTING.md)
- **版本历史**：参见 [CHANGELOG.md](CHANGELOG.md)

---

## 许可

MIT License
