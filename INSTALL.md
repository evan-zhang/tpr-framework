# TPR Framework 安装指南

> 本文档面向需要安装和配置 TPR Framework 的 AI Agent。

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

## 二、AGENTS.md 配置

在当前 Agent 的 `AGENTS.md` 中添加：

```yaml
tpr_mode: full          # 使用 TPR 全流程模式
can_spawn: true         # 必须为 true（需要 sub-agent 能力）

# RT 项目根目录（本地 RT 目录所在位置）
rt_root_dir: "~/.openclaw/gateways/life/domains/{agent-id}/workspace/projects"
```

| 配置项 | 必填 | 默认值 | 说明 |
|--------|------|--------|------|
| `tpr_mode` | 是 | — | 填 `full` |
| `can_spawn` | 是 | — | 必须为 `true` |
| `rt_root_dir` | 是 | — | 本地 RT 项目目录的根路径 |

---

## 三、更新 TPR Skill

```bash
cd ~/.openclaw/skills/tpr-framework
git pull origin main
```

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

## 五、快速验证

对 Agent 说：

> "用 TPR 分析一下 XXX 需求"

Agent 应该按 DISCOVERY → GRV → Battle → Implementation 流程执行，产出文件写入 `rt_root_dir` 下的项目目录。

---

## 常见问题

| 问题 | 解决 |
|------|------|
| Agent 说"不具备 sub-agent 能力" | 检查 `can_spawn: true` 是否已配置 |
