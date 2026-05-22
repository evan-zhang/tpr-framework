# 安装指引

## 项目地址

- **TPR Framework**：<https://github.com/evan-zhang/tpr-framework>
- **Ralph Loop**（持续执行引擎）：<https://github.com/evan-zhang/agent-factory>

## 安装

```bash
# 安装 TPR Framework
git clone https://github.com/evan-zhang/tpr-framework.git ~/.openclaw/skills/tpr-framework

# 安装 Ralph Loop（强烈推荐）
git clone https://github.com/evan-zhang/agent-factory.git
```

完整的安装与使用指南见项目 [README.md](https://github.com/evan-zhang/tpr-framework#readme)。

## 运行时检测

TPR Framework 不需要手动配置任何参数。安装后即可使用。

| 检测项 | 方法 |
|--------|------|
| sub-agent 能力 | 检查运行环境是否支持 spawn sub-agent |
| TPR 模式 | 根据判定矩阵自动判定（全流程 / TPR 思维） |
| RT 根目录 | 首次使用时向用户建议默认路径，用户确认后使用 |
