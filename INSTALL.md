# TPR Framework 安装指南

> 本文档是面向 AI Agent 的快速安装卡片。完整的安装与使用指南见 [README.md](README.md)。

---

## 安装

### 1. TPR Framework

```bash
git clone https://github.com/evan-zhang/tpr-framework.git ~/.openclaw/skills/tpr-framework
```

### 2. Ralph Loop（强烈推荐）

> ⚠️ 不装 Ralph Loop，Implementation 阶段缺乏持续验证。详见 [README.md](README.md)。

```bash
# 推荐方式：克隆 agent-factory 仓库
git clone https://github.com/evan-zhang/agent-factory.git
# Ralph Loop 位于 agent-factory/projects/2605211/ralph/
```

或单独下载关键文件（详见 [README.md](README.md) 第二步）。

### 3. 验证

```bash
# TPR Framework
ls ~/.openclaw/skills/tpr-framework/SKILL.md

# Ralph Loop
bash ralph/scripts/ralph-loop.sh --help
```

---

## 运行时检测（无需配置）

TPR Framework 不需要手动配置任何参数。以下全部由 Agent 在运行时自动完成：

| 检测项 | 方法 |
|--------|------|
| sub-agent 能力 | 检查运行环境是否支持 spawn |
| TPR 模式 | 根据判定矩阵自动判定 |
| RT 根目录 | 首次使用时向用户建议默认路径，用户可指定 |

---

## 更新

```bash
cd ~/.openclaw/skills/tpr-framework && git pull origin main
cd agent-factory && git pull origin master
```
