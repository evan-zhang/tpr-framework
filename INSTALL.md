# TPR Framework 安装指南

> 本文档面向需要安装和配置 TPR Framework 的 AI Agent。

---

## 前提条件

| 依赖 | 要求 |
|------|------|
| OpenClaw | 已安装并正常运行 |
| Git | 已安装，能访问 GitHub |
| Node.js | >= 18（用于 openclaw-xgkb-sync 同步服务） |
| openclaw-xgkb-sync | 已部署（见下方） |

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

# 同步服务地址（可选，用于查询真实同步状态）
sync_service_url: "http://127.0.0.1:9090"
```

| 配置项 | 必填 | 默认值 | 说明 |
|--------|------|--------|------|
| `tpr_mode` | 是 | — | 填 `full` |
| `can_spawn` | 是 | — | 必须为 `true` |
| `rt_root_dir` | 是 | — | 本地 RT 项目目录的根路径 |
| `sync_service_url` | 否 | `http://127.0.0.1:9090` | 同步服务 HTTP API 地址 |

---

## 三、部署 openclaw-xgkb-sync 同步服务

### 3.1 克隆并安装

```bash
gh repo clone xgjk/openclaw-xgkb-sync /path/to/openclaw-xgkb-sync
cd /path/to/openclaw-xgkb-sync
npm install
```

### 3.2 配置 config.json

```bash
cp config.example.json config.json
```

编辑 `config.json`，添加一条全局 mapping 覆盖整个 projects 目录：

```json
{
  "serverUrl": "https://sg-al-cwork-web.mediportal.com.cn/open-api/",
  "appKey": "<你的-appKey>",
  "syncDirection": "bidirectional",
  "autoSyncIntervalSec": 120,
  "stateDbPath": "./openclaw-sync-state.db",
  "managementPort": 9090,
  "managementHost": "127.0.0.1",
  "mappings": [
    {
      "mappingId": "tpr-projects",
      "enabled": true,
      "localRoot": "/absolute/path/to/projects",
      "remoteRootFolderPath": "TPR",
      "filePatterns": ["**/*.md"],
      "excludePatterns": ["**/.git/**"]
    }
  ]
}
```

| 字段 | 说明 |
|------|------|
| `serverUrl` | 玄关知识库 API 地址 |
| `appKey` | 玄关开放平台 API 密钥 |
| `localRoot` | 本地 projects 目录的**绝对路径**（替换为实际路径） |
| `remoteRootFolderPath` | 知识库中存放 TPR 项目的根目录 |

### 3.3 启动并验证

```bash
node dist/index.js --config config.json &
sleep 3
curl http://127.0.0.1:9090/health
```

正常返回：

```json
{"ok":true,"version":"1.0.0","pid":12345,...}
```

验证 mapping 已注册：

```bash
curl http://127.0.0.1:9090/mappings
```

### 3.4 设置开机自启

**macOS（launchd）**：写入 `~/Library/LaunchAgents/com.openclaw.xgkb-sync.plist`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.openclaw.xgkb-sync</string>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/node</string>
    <string>/path/to/openclaw-xgkb-sync/dist/index.js</string>
    <string>--config</string>
    <string>/path/to/openclaw-xgkb-sync/config.json</string>
  </array>
  <key>WorkingDirectory</key>
  <string>/path/to/openclaw-xgkb-sync</string>
  <key>RunAtLoad</key>
  <true/>
  <key>KeepAlive</key>
  <true/>
  <key>StandardOutPath</key>
  <string>/path/to/openclaw-xgkb-sync/sync.log</string>
  <key>StandardErrorPath</key>
  <string>/path/to/openclaw-xgkb-sync/sync.error.log</string>
</dict>
</plist>
```

加载：`launchctl load ~/Library/LaunchAgents/com.openclaw.xgkb-sync.plist`

**Linux（systemd）**：写入 `/etc/systemd/system/openclaw-xgkb-sync.service`：

```ini
[Unit]
Description=OpenClaw XGKB Sync Agent
After=network.target

[Service]
WorkingDirectory=/path/to/openclaw-xgkb-sync
ExecStart=/usr/bin/node dist/index.js --config config.json
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

启动：`sudo systemctl daemon-reload && sudo systemctl enable --now openclaw-xgkb-sync`

---

## 四、工作机制

TPR 和同步服务完全解耦：

```
TPR 编排者写入本地文件 → openclaw-xgkb-sync 每 120s 自动同步 → 知识库
```

编排者**不参与任何同步操作**，只管写文件。同步服务自动处理推送、拉取、冲突（LWW）。

### 查询真实同步状态

TPR 可以随时调用以下接口获取真实状态：

```bash
# 查询服务健康状态
curl http://127.0.0.1:9090/health

# 查询当前所有 mapping
curl http://127.0.0.1:9090/mappings

# 触发立即同步（非强制）
curl -X POST http://127.0.0.1:9090/sync
```

---

## 五、更新 TPR Skill

```bash
cd ~/.openclaw/skills/tpr-framework
git pull origin main
```

---

## 六、快速验证

对 Agent 说：

> "用 TPR 分析一下 XXX 需求"

Agent 应该按 DISCOVERY → GRV → Battle → Implementation 流程执行，产出文件写入 `rt_root_dir` 下的项目目录，同步服务自动推送。

---

## 常见问题

| 问题 | 解决 |
|------|------|
| Agent 说"不具备 sub-agent 能力" | 检查 `can_spawn: true` 是否已配置 |
| 同步服务连接失败 | `curl http://127.0.0.1:9090/health` 验证服务是否运行 |
| 知识库没有文件 | 检查 `localRoot` 是否为绝对路径，`mappings` 是否已注册 |
| mapping 注册成功但不同步 | 等待 `autoSyncIntervalSec`（默认 120s）或手动触发 `/sync` |
