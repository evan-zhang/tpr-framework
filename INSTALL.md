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

## 二、快速验证

对 Agent 说：

> "用 TPR 分析一下 XXX 需求"

Agent 应该按 DISCOVERY → GRV → Battle → Implementation 流程执行。

---

## 三、运行时检测说明

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

## 四、（可选）安装 Ralph Loop

如果需要使用 Phase 4 Mode B（Ralph Loop 持续执行），需额外安装 Ralph Loop：

```bash
# 方式一：克隆 agent-factory 仓库
git clone https://github.com/evan-zhang/agent-factory.git
# Ralph Loop 位于 agent-factory/projects/2605211/ralph/

# 方式二：单独下载
curl -fsSL https://raw.githubusercontent.com/evan-zhang/agent-factory/master/projects/2605211/ralph/SKILL.md -o ralph/SKILL.md

# 详见 references/tpr-bridge-protocol.md § Ralph Loop 安装
```

---

## 五、更新 TPR Skill

```bash
cd ~/.openclaw/skills/tpr-framework
git pull origin main
```

---

## 常见问题

| 问题 | 解决 |
|------|------|
| Agent 说"不具备 sub-agent 能力" | 当前运行环境不支持 spawn，自动降级为 TPR 思维模式 |
| RT 文件写到了哪里 | 检查 Agent 建议的 RT 根目录，默认是 `{workspace}/projects` |
