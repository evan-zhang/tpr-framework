# 安装与知识库同步

## 项目地址

- **TPR Framework**：<https://github.com/evan-zhang/tpr-framework>
- **Ralph Loop**：<https://github.com/evan-zhang/agent-factory>（TPR 持续执行引擎，强烈推荐安装）
- **xgkb-sync-helper**：<https://github.com/evan-zhang/xgkb-sync-helper>（知识库同步助手，推荐安装）

## 快速安装

```bash
# 安装 TPR Framework
git clone https://github.com/evan-zhang/tpr-framework.git ~/.openclaw/skills/tpr-framework

# 安装 Ralph Loop（强烈推荐）
git clone https://github.com/evan-zhang/agent-factory.git

# 安装知识库同步助手（推荐）
git clone https://github.com/evan-zhang/xgkb-sync-helper.git ~/.openclaw/skills/xgkb-sync-helper
```

完整安装与使用指南见项目 [README.md](https://github.com/evan-zhang/tpr-framework#readme)。

## 知识库同步

TPR 项目产出自动同步到玄关个人知识库，实现公网预览。

### 配置（一次性）

1. 全局 appKey：
```bash
cat > ~/.openclaw/xgkb.json << 'EOF'
{"appKey": "你的玄关开放平台 appKey", "serverUrl": "https://sg-al-cwork-web.mediportal.com.cn/open-api/"}
EOF
```

2. 项目集合根目录（`projects/`）放一个 `.xgkb.json`，所有项目共用：
```json
{"enabled": true, "remoteRoot": "TPR-Framework"}
```

### 同步规则

编排者或 sub-agent 每次用 write/edit 写入文件后，追加一步：
```bash
python3 ~/.openclaw/skills/xgkb-sync-helper/scripts/xgkb_push.py <文件路径>
```

- fire-and-forget，失败不阻断主流程
- 文本文件（.md/.txt/.json 等）和二进制文件（.pdf/.docx 等）均支持幂等同步
- 无配置或 `enabled: false` 时静默跳过
- 批量同步整个目录：`python3 ~/.openclaw/skills/xgkb-sync-helper/scripts/xgkb_sync_dir.py <目录>`
