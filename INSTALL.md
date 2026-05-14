# TPR Framework 安装指南

> 本文档面向需要安装和配置 TPR Framework 的 AI Agent。

---

## 前提条件

| 依赖 | 要求 |
|------|------|
| OpenClaw | 已安装并正常运行 |
| Git | 已安装，能访问 GitHub |
| Node.js | >= 18（用于 openclaw-xgkb-sync 同步服务） |
| openclaw-xgkb-sync | 已部署（见下方「同步服务部署」） |

---

## 一、安装 TPR Framework Skill

### 方式一：Git Clone（推荐）

```bash
# 克隆到 OpenClaw skills 目录
git clone https://github.com/evan-zhang/tpr-framework.git ~/.openclaw/skills/tpr-framework
```

### 方式二：从本地工作区同步

如果代码已在本地工作区：

```bash
# 确认本地路径
ls ~/.openclaw/skills/tpr-framework/SKILL.md
# 或
ls /path/to/your/workspace/projects/tpr-framework/SKILL.md
```

---

## 二、AGENTS.md 配置

在当前 Agent 的 `AGENTS.md` 中添加以下配置：

```yaml
# TPR Framework
tpr_mode: full           # 使用 TPR 全流程模式
can_spawn: true          # 必须为 true（需要 sub-agent 能力）

# RT 项目根目录（本地 RT 目录所在位置）
rt_root_dir: "~/.openclaw/gateways/life/domains/{agent-id}/workspace/projects"

# TPR 知识库同步配置（可选，要启用同步才填）
tpr_config:
  kb_sync: true                          # true = 启用同步
  sync_service_url: "http://127.0.0.1:9090"  # openclaw-xgkb-sync 服务地址
  kb_root_folder: "TPR"                  # 知识库根目录
```

| 配置项 | 必填 | 默认值 | 说明 |
|--------|------|--------|------|
| `tpr_mode` | 是 | — | 填 `full` 表示使用 TPR 全流程 |
| `can_spawn` | 是 | — | 必须为 `true`，TPR 全流程需要 sub-agent |
| `rt_root_dir` | 是 | — | 本地 RT 项目目录的根路径 |
| `kb_sync` | 否 | `false` | 是否启用知识库同步 |
| `sync_service_url` | 否 | `http://127.0.0.1:9090` | openclaw-xgkb-sync HTTP API |
| `kb_root_folder` | 否 | `TPR` | 知识库中存放 TPR 项目的根目录 |

---

## 三、openclaw-xgkb-sync 同步服务部署（可选）

如果 `kb_sync: true`，必须先部署同步服务。

### 3.1 克隆同步服务

```bash
gh repo clone xgjk/openclaw-xgkb-sync /path/to/openclaw-xgkb-sync
cd /path/to/openclaw-xgkb-sync
npm install
```

### 3.2 创建配置文件

```bash
cp config.example.json config.json
```

编辑 `config.json`，填入以下必填项：

```json
{
  "serverUrl": "https://sg-al-cwork-web.mediportal.com.cn/open-api/",
  "appKey": "<你的-appKey>",
  "syncDirection": "bidirectional",
  "autoSyncIntervalSec": 120,
  "stateDbPath": "./openclaw-sync-state.db",
  "managementPort": 9090,
  "managementHost": "127.0.0.1",
  "mappings": []
}
```

| 字段 | 说明 |
|------|------|
| `serverUrl` | 玄关知识库 API 地址：`https://sg-al-cwork-web.mediportal.com.cn/open-api/` |
| `appKey` | 玄关开放平台签发的 API 密钥 |
| `mappings` | 留空，由 TPR Skill 在运行时动态注册 |

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

### 3.4 设置开机自启（macOS / Linux）

**macOS（launchd）**：将以下 plist 写入 `~/Library/LaunchAgents/com.openclaw.xgkb-sync.plist`：

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

## 四、TPR Skill 更新

Skill 有更新时，执行：

```bash
cd ~/.openclaw/skills/tpr-framework
git pull origin main
```

---

## 五、快速验证

安装完成后，对 Agent 说一句话验证：

> "用 TPR 分析一下 XXX 需求"

Agent 应该：
1. 自动激活 TPR Framework Skill
2. 按 DISCOVERY → GRV → Battle → Implementation 流程执行
3. 如 `kb_sync: true`，新项目激活时会自动注册 mapping 到同步服务

---

## 常见问题

| 问题 | 原因 | 解决 |
|------|------|------|
| Agent 说"不具备 sub-agent 能力" | `can_spawn` 未设为 `true` | 检查 AGENTS.md 配置 |
| 同步服务连接失败 | 服务未启动或端口错误 | `curl http://127.0.0.1:9090/health` 验证 |
| 知识库没有文件 | mapping 未注册 | 新项目激活时会自动注册，检查 `/mappings` |
| `kb_sync: true` 但不生效 | 配置缺少必填项 | 确认 `sync_service_url` 和 `kb_root_folder` 已填 |
