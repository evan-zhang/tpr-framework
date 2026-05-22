# TPR Framework 安装指南

> 本文档面向需要安装 TPR Framework 的 AI Agent。

---

## 前提条件

| 依赖 | 要求 |
|------|------|
| OpenClaw | 已安装并正常运行 |
| Git | 已安装，能访问 GitHub |

---

## 一、安装 TPR Framework Skill

```bash
git clone https://github.com/evan-zhang/tpr-framework.git ~/.openclaw/skills/tpr-framework
```

或从本地已存在的仓库更新：

```bash
cd ~/.openclaw/skills/tpr-framework && git pull origin main
```

---

## 二、安装 Ralph Loop（TPR 持续执行引擎）

> ⚠️ **TPR + Ralph Loop 才是完整方案。** TPR 负责从需求到方案（DISCOVERY → GRV → Battle），Ralph Loop 负责从方案到交付（Implementation）。不装 Ralph Loop，相当于 Battle 通过后缺了一条腿——方案定了但没人持续执行和验证。

Ralph Loop 提供循环验证机制，确保每一步执行都经过自检，避免"做到一半发现方案有问题"的风险。

### 前置要求

- bash（macOS / Linux 自带）
- AI 执行器（任选其一）：[Claude Code](https://docs.anthropic.com/en/docs/claude-code) 或 [OpenAI Codex](https://github.com/openclaw/codex)
- git

### 安装方式

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

### 验证安装

```bash
bash ralph/scripts/ralph-loop.sh --help
bash ralph/scripts/init-state.sh --help
```

详细的桥接规范见 `references/tpr-bridge-protocol.md`。

---

## 三、快速验证

对 Agent 说：

> "用 TPR 分析一下 XXX 需求"

Agent 应该按 DISCOVERY → GRV → Battle → Implementation 流程执行。

---

## 四、运行时检测说明

TPR Framework 不需要手动配置。Agent 在运行时自动完成以下检测：

| 检测项 | 方法 | 影响 |
|--------|------|------|
| sub-agent 能力 | 检查运行环境是否支持 spawn（如 `sessions_spawn`、`agent` 工具等） | 不具备则降级为 TPR 思维 |
| TPR 模式 | 根据判定矩阵（A/B/C/D 四项）自动判定 | A/B/C ≥ 2 → 全流程，否则 → TPR 思维 |
| RT 根目录 | 首次使用时向用户建议默认路径（`{workspace}/projects`），用户可指定其他路径 | 用户确认后在整个流程中使用 |

### RT 根目录确认流程

1. Agent 首次启动 TPR 流程时，向用户建议：`{workspace}/projects`
2. 用户确认，或指定新路径
3. 确认后的路径在当前会话中持续使用
4. 用户随时可以通过对话重新指定

---

## 五、更新

```bash
# 更新 TPR Framework
cd ~/.openclaw/skills/tpr-framework && git pull origin main

# 更新 Ralph Loop（方式一安装的）
cd agent-factory && git pull origin master
```

---

## 常见问题

| 问题 | 解决 |
|------|------|
| Agent 说"不具备 sub-agent 能力" | 当前运行环境不支持 spawn，自动降级为 TPR 思维模式 |
| RT 文件写到了哪里 | 检查 Agent 建议的 RT 根目录，默认是 `{workspace}/projects` |
| 没装 Ralph Loop 会怎样 | Implementation 阶段自动降级为 Mode A（单线程）或 Mode C（一次性执行），功能可用但缺乏持续验证。强烈建议安装 |
