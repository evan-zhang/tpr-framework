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

### 3. xgkb-sync-helper（推荐）

> 安装后 TPR 产出的文件会自动同步到玄关知识库，支持公网预览。不装则只写本地磁盘。

```bash
git clone https://github.com/evan-zhang/xgkb-sync-helper.git ~/.openclaw/skills/xgkb-sync-helper
```

配置 appKey：

```bash
cat > ~/.openclaw/xgkb.json << 'EOF'
{
  "appKey": "你的玄关开放平台 appKey",
  "serverUrl": "https://sg-al-cwork-web.mediportal.com.cn/open-api/"
}
EOF
```

在 TPR 项目根目录启用同步：

```bash
echo '{"enabled": true, "remoteRoot": "TPR-Framework"}' > /path/to/your/project/.xgkb.json
```

> appKey 获取：玄关开放平台 → 个人设置 → API 密钥

### 4. 验证

```bash
# TPR Framework
ls ~/.openclaw/skills/tpr-framework/SKILL.md

# xgkb-sync-helper
python3 ~/.openclaw/skills/xgkb-sync-helper/scripts/xgkb_push.py --help

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
cd ~/.openclaw/skills/xgkb-sync-helper && git pull origin main
cd agent-factory && git pull origin master
```
