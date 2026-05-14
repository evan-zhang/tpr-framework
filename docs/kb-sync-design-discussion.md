# 知识库同步方案讨论 — 现状、需求与路线选择

> **文档性质**：讨论稿（已重大更新）
> **创建**：2026-05-14
> **更新**：2026-05-14（基于 openclaw-xgkb-sync 新发现）
> **参与者**：Evan & Codex

---

## 一、现状

### 1.1 用户环境

| 设备 | 数量 | 操作系统 |
|------|------|---------|
| 台式电脑 | 1 | macOS |
| 笔记本电脑 | 1 | macOS |
| 手机 | 1 | iOS / Android |

**核心需求**：三台设备的 Markdown 文档在所有设备上保持同步，共享同一份知识库。

### 1.2 现有工具链

我们已有**两套**同步工具，分别针对不同场景：

#### 工具 A：openclaw-xgkb-sync（核心同步引擎）

> 仓库：https://github.com/xgjk/openclaw-xgkb-sync

**定位**：独立 Node.js 后台服务，不依赖 Obsidian 客户端，直接作为文件系统同步 Agent 运行。

| 能力 | 实现 |
|------|------|
| 同步方向 | bidirectional / push / pull 三种模式 |
| 同步算法 | LWW（Last-Write-Wins），基于 mtime |
| 增量同步 | listChanges API + SQLite 水位标记，零变化时毫秒级完成 |
| 删除同步 | 双向传播（可配置） |
| 多 mapping | 单节点可配置多条本地目录 ↔ 云端目录映射，每条独立方向 |
| 限速 | 令牌桶，每个 appKey 独立计算，互不干扰 |
| 管理 API | HTTP 内置：health / status / sync / reload / mapping CRUD |
| 部署 | systemd（Linux）/ launchd（macOS）常驻进程 |
| 状态持久化 | SQLite WAL 模式，状态不丢失 |
| 热重载 | config.json 变更自动 reload，无需重启进程 |

**架构**：

```
src/
├── index.ts           入口：参数解析 + 配置加载 + 调度器启动
├── scheduler.ts       多 mapping 调度（防重入、按 appKey 限速、并发控制）
├── syncEngine.ts      核心同步决策与执行（LWW 策略）
├── syncStateDb.ts     SQLite 状态库（增量水位）
├── remoteFs.ts       云端文件系统适配器
├── localFs.ts        本地文件系统适配器
├── kbApi.ts          知识库 HTTP 客户端（Node fetch + 重试 + 限速）
├── rateLimiter.ts    令牌桶限速器
├── managementApi.ts   HTTP 管理 API
└── config.ts         配置加载与校验
```

#### 工具 B：obsidian-xgkb-sync（Obsidian 插件）

> 仓库：待查（workspace 中已有实现）

**定位**：Obsidian 插件形态的双向同步，面向习惯使用 Obsidian GUI 的用户。

| 能力 | 说明 |
|------|------|
| 同步方向 | bidirectional / push / pull |
| 增量检测 | SQLite SyncStateDb |
| 删除同步 | 双向删除传播 |
| 自动定时 | 可配置分钟级间隔 |
| 移动端 | 支持 Obsidian Mobile |

**依赖**：Obsidian API（`App`、`TFile`、`requestUrl` 等），只能跑在 Obsidian 环境内。

### 1.3 工具对比与选择

| 场景 | 工具 A（openclaw-xgkb-sync） | 工具 B（obsidian-xgkb-sync） |
|------|------|------|
| 桌面 Mac/PC（纯后台同步） | ✅ **首选** | ⚠️ 需要开着 Obsidian |
| 桌面（需要 Obsidian 编辑） | ✅ + Obsidian 并存 | ✅ |
| 移动端（iOS/Android） | ❌ 无法运行 CLI | ✅ Obsidian Mobile + 插件 |
| 服务器环境 | ✅ 纯 Node.js | ❌ 需要图形界面 |
| 多用户 / 多目录 | ✅ 多 mapping | ❌ 单一目录 |

**结论**：
- **桌面主力**：部署 `openclaw-xgkb-sync` 作为后台服务
- **移动端**：Obsidian Mobile + obsidian-xgkb-sync 插件
- **两者并存也完全兼容**：都指向同一个知识库，最终殊途同归

---

## 二、需求分析

### 2.1 核心需求

| 需求 | 说明 |
|------|------|
| 多设备文档同步 | 两台电脑 + 手机，文档在所有设备上一致 |
| AI 可读写 | Agent 能将产出推送到知识库，也能从知识库读取文件 |
| 用户可管理 | 用户可以通过 GUI 浏览、上传、管理知识库中的文件 |
| 移动端友好 | iOS/Android 操作门槛低，最好全自动 |

### 2.2 不同角色的需求

| 角色 | 场景 | 推荐工具 |
|------|------|---------|
| **用户（桌面，后台同步）** | 希望文档自动同步，不需要开着 Obsidian | openclaw-xgkb-sync 服务 |
| **用户（桌面，Obsidian 编辑）** | 习惯 Obsidian GUI，同时需要同步 | obsidian-xgkb-sync 插件 |
| **用户（移动）** | 手机上查看/编辑文档 | Obsidian Mobile + 插件 |
| **用户（知识库 GUI）** | 在 Web 界面上传/管理文件 | 知识库原生 GUI |
| **AI Agent（TPR）** | 将 TPR 产出推送到知识库、从知识库读取项目 | openclaw-xgkb-sync 服务（文件落地本地后自动同步） |

---

## 三、架构设计（推荐方案）

### 3.1 最终架构

```
┌──────────────────────────────────────────────────────────────────┐
│                      玄关知识库（统一存储，单一真相源）               │
└─────────────────────────────┬──────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
   ┌────▼────┐          ┌────▼────┐          ┌────▼────┐
   │ Mac Mini │          │ MacBook  │          │  手机   │
   │          │          │          │          │         │
   │ openclaw │          │ openclaw │          │ Obsidian│
   │  -xgkb-  │◄────────►│  -xgkb-  │◄────────►│ Mobile  │
   │  sync    │  WiFi/   │  sync    │  WiFi/   │ +插件   │
   │ (后台)   │  4G      │ (后台)   │  4G      │         │
   │          │          │          │          │         │
   │  TPR     │          │  TPR     │          │         │
   │ 产出文件 │          │ 产出文件 │          │         │
   └──────────┘          └──────────┘          └─────────┘

   TPR Skill：只需把产出写到本地目录
   openclaw-xgkb-sync：自动将文件同步到知识库
   用户从知识库 GUI 上传的文件：自动同步到本地
```

### 3.2 TPR 与同步的边界

**关键结论：TPR 不需要任何同步逻辑。**

TPR 的产出文件写到本地目录（例如 `~/workspace/projects/xxx/`），`openclaw-xgkb-sync` 后台服务自动将文件推送到知识库。整个过程对 TPR 完全透明。

**TPR output-delivery.md 改动方向**：
- 删除所有同步实现细节（API 调用、版本管理、索引机制）
- 简化为：**"把文件写到本地目录 X，同步服务会自动处理"**
- 新增配置声明：本地哪个目录对应知识库的哪个路径

### 3.3 配置示例

每台设备的 `config.json`：

```json
{
  "serverUrl": "https://sg-al-cwork-web.mediportal.com.cn/open-api/",
  "appKey": "TsFhRR7OywNULeHPqudePf85STc4EpHI",
  "syncDirection": "bidirectional",
  "autoSyncIntervalSec": 120,
  "stateDbPath": "~/.openclaw/xgkb-sync-state.db",
  "managementPort": 9090,
  "mappings": [
    {
      "mappingId": "workspace",
      "localRoot": "/Users/evan/.openclaw/gateways/life/domains/aodw_codex/workspace",
      "remoteRootFolderPath": "OpenClaw/workspace",
      "syncDirection": "bidirectional",
      "filePatterns": ["**/*.md"],
      "excludePatterns": ["**/.git/**", "**/node_modules/**"]
    }
  ]
}
```

---

## 四、与 Obsidian 的关系

### 4.1 三种用户场景

| 场景 | 用户行为 | 同步工具 |
|------|---------|---------|
| **纯知识管理** | 所有设备用知识库 GUI 或 Obsidian GUI | obsidian-xgkb-sync 插件 |
| **AI 工作流** | Agent（TPR）产出写到 workspace，本地 Obsidian 查看 | openclaw-xgkb-sync 服务 |
| **混合** | 日常用 Obsidian，AI 工作流产出也在 Obsidian Vault 里 | 两者并存 |

### 4.2 两者并存是否冲突？

**不冲突。** 前提是目录错开：

| 场景 | Obsidian Vault 目录 | openclaw-xgkb-sync 同步目录 |
|------|------|------|
| 完全分离 | `/Users/evan/obsidian-vault/` | `/Users/evan/workspace/` |
| 有重叠（Vault 是 workspace 子集） | — | 需确保 filePatterns 错开 |

**推荐完全分离**：Obsidian Vault 只管理个人笔记，AI 产出放在 workspace 独立目录。

### 4.3 移动端方案

移动端（iOS/Android）无法运行 `openclaw-xgkb-sync`（Node.js CLI），但有两条可行路径：

| 方案 | 说明 |
|------|------|
| **Obsidian Mobile + 插件** | 安装 Obsidian Mobile + obsidian-xgkb-sync，配置同 knowledge base 连接，插件自动同步 |
| **知识库 GUI** | 直接用浏览器访问知识库 Web 界面浏览/上传，无法自动同步到本地文件 |

**推荐**：日常笔记用 Obsidian Mobile + 插件；AI 工作流产出（workspace）在桌面设备使用，移动端通过知识库 GUI 按需查阅。

---

## 五、TPR 改造计划

### 5.1 output-delivery.md 精简方向

**删除内容**：
- 所有知识库 API 调用细节（uploadContent、getFullFileContent 等）
- kb-registry.yaml 文件 ID 映射机制
- 云端目录索引（index.yaml）
- 同步模式配置（push_only / push_pull / manual）
- 文件格式转换规则（文本文件自动转 MD）
- 从知识库恢复项目的流程

**保留内容**：
- 阶段完成时的文件命名和目录结构规范（这是 TPR 自己的交付契约）
- 配置声明（`tpr_config`，但简化为本地目录和云端路径的映射关系）
- 一句话：**"写到本地目录，同步服务自动处理"**

### 5.2 新配置项

```yaml
# AGENTS.md 中
tpr_config:
  kb_sync: true                    # 是否启用知识库同步
  local_output_root: "~/workspace/projects"  # TPR 产出的本地根目录
  remote_folder: "TPR"             # 对应知识库的远端目录路径
```

### 5.3 输出物清单

| 文件 | 写到本地 | 同步到知识库 |
|------|---------|-------------|
| DISCOVERY.md | ✅ | ✅ |
| GRV.md | ✅ | ✅ |
| BATTLE-*.md | ✅ | ✅ |
| 交付物.md | ✅ | ✅ |
| P-ACPT.md | ✅ | ✅ |
| 用户上传的参考文件 | ✅ | ✅ |

---

## 六、openclaw-xgkb-sync 部署计划

### 6.1 部署清单

| 设备 | 部署方式 | 状态 |
|------|---------|------|
| Mac Mini（主设备） | launchd 常驻 | 待部署 |
| MacBook（笔记本） | launchd 常驻 | 待部署 |
| 手机（iOS/Android） | Obsidian Mobile + 插件 | 待配置 |

### 6.2 待验证事项

| # | 问题 | 优先级 |
|---|------|--------|
| 1 | openclaw-xgkb-sync 在 Mac Mini 上实际运行情况 | 高 |
| 2 | obsidian-xgkb-sync 插件最新代码在哪里（workspace 里的 obsidian-xgkb-sync 是否是最新版） | 高 |
| 3 | 移动端 Obsidian Mobile 插件同步体验 | 中 |
| 4 | 多 mapping 场景：workspace 和 Obsidian Vault 目录如何安全共存 | 中 |
| 5 | `openclaw-xgkb-sync` 的 `excludePatterns` 是否能正确过滤 .git / node_modules | 低 |

---

## 七、结论与行动项

### 7.1 明确结论

1. **同步已解决**：openclaw-xgkb-sync 是生产级方案，不需要再发明轮子
2. **TPR 与同步解耦**：TPR 只需写本地文件，同步服务自动处理
3. **移动端**：Obsidian Mobile + obsidian-xgkb-sync 插件是唯一可行路径
4. **obsidian-xgkb-sync 插件仍有价值**：面向习惯 Obsidian GUI 的用户，两者并存不冲突
5. **不需要创建 kb-sync Skill**：openclaw-xgkb-sync 就是同步工具本身

### 7.2 下一步行动

| # | 行动 | 负责方 |
|---|------|--------|
| A | Mac Mini 上部署 openclaw-xgkb-sync 服务 | 待 Evan 确认 |
| B | MacBook 上部署 openclaw-xgkb-sync 服务 | 待 Evan 确认 |
| C | 精简 TPR output-delivery.md（删除所有同步实现细节） | Codex |
| D | 更新 TPR SKILL.md 同步配置声明 | Codex |
| E | obsidian-xgkb-sync 插件代码确认真在哪个仓库 | 待确认 |
| F | 移动端 Obsidian Mobile + 插件配置测试 | 待 Evan 确认 |

---

*本文档为讨论稿，待确认后进入实施。*
