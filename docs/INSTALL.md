# TPR Framework — Agent 安装配置指南

> 本文档面向 **Agent 管理员**，指导如何为你的 AI Agent 安装和配置 TPR Framework Skill。

---

## 一、安装 Skill

### 1.1 克隆到 Skills 目录

```bash
cd ~/.openclaw/skills/
git clone https://github.com/evan-zhang/tpr-framework.git
```

### 1.2 验证安装

```bash
ls ~/.openclaw/skills/tpr-framework/SKILL.md
# 能看到文件即安装成功
```

Agent 会在下次对话时自动识别并加载这个 Skill。无需重启。

### 1.3 后续升级

```bash
cd ~/.openclaw/skills/tpr-framework/
git pull origin main
```

---

## 二、基础配置（可选但推荐）

在你的 Agent 的 `AGENTS.md` 中声明 TPR 使用模式：

```yaml
# TPR 基础配置
tpr_mode: full          # full = 全流程（需 can_spawn=true）, cognitive = 纯思维模式
can_spawn: true         # 是否能派生 sub-agent（full 模式必须为 true）
```

不配置也能用——Agent 默认进入 TPR 思维模式（不需要 sub-agent）。

---

## 三、知识库同步配置（可选）

TPR 支持将产出文件自动同步到玄关知识库。需要以下配置：

### 3.1 获取 appKey

联系知识库管理员获取你的 appKey。

### 3.2 设置环境变量

**关键原则：appKey 只存环境变量，永不硬编码到配置文件中。**

```bash
# 将 appKey 写入 shell 配置（替换为你自己的 key）
echo 'export KB_APP_KEY="your-appkey-here"' >> ~/.zshrc
source ~/.zshrc
```

### 3.3 在 Agent 配置中声明

在 Agent 的 `AGENTS.md` 中加入：

```yaml
# TPR 知识库同步配置
tpr_config:
  kb_sync: true              # 启用知识库同步（默认 false）
  kb_appkey_env: KB_APP_KEY  # 存放 appKey 的环境变量名
  kb_root_folder: "TPR"      # 知识库中的根目录名
  kb_project_id: null         # null = 个人知识库，或填项目空间 ID
```

### 3.4 验证连通性

```bash
curl -s -X GET \
  'https://sg-al-cwork-web.mediportal.com.cn/open-api/document-database/file/getChildFiles?parentId=0&pageSize=5' \
  -H "appKey: $KB_APP_KEY"
# 返回 {"resultCode":1,...} 表示连通成功
```

### 3.5 配置项说明

| 配置项 | 必填 | 默认值 | 说明 |
|--------|------|--------|------|
| kb_sync | 否 | false | true = 每阶段完成后自动同步知识库 |
| kb_appkey_env | 是* | - | 环境变量名（kb_sync=true 时必填） |
| kb_root_folder | 否 | "TPR" | 知识库根目录，支持多级如 "项目文档/TPR" |
| kb_project_id | 否 | null | 项目空间 ID，null = 个人知识库 |

\* 不配置 kb_sync 或设为 false 时，TPR 只写本地文件，不依赖知识库。

---

## 四、知识库同步效果

启用后，TPR 产出会自动同步到知识库：

```
TPR/{项目编号}/
├── 01-discovery/
│   ├── DISCOVERY.md          ← AI 消费（RAG 索引）
│   └── DISCOVERY.html        ← 人消费（可视化预览）
├── 02-planning/
│   ├── GRV.md
│   └── GRV.html
├── 03-battle/
│   └── BATTLE-*.md           ← 过程记录（只有 MD）
├── 04-execution/
│   ├── 交付物.md
│   └── 交付物.html
├── 05-closure/
│   ├── P-ACPT.md
│   └── P-ACPT.html
└── kb-registry.yaml          ← 文件 ID 映射表
```

**多版本管理**：同一文件修改后自动追加新版本（V1.0 → V2.0 → ...），路径和 fileId 不变。

---

## 五、安全降级

任何配置问题都不会阻塞 TPR 主流程：

| 场景 | 行为 |
|------|------|
| 未配置 tpr_config | 只写本地文件，正常工作 |
| kb_sync = false | 只写本地文件 |
| appKey 无效或环境变量为空 | 报告错误，降级为本地模式 |
| 知识库 API 不可达 | 报告错误，降级为本地模式 |

---

## 六、快速验证

安装配置完成后，可用以下测试验证：

### 测试 1：Skill 加载

> 问 Agent："TPR 全流程中，策划层、审查层、执行层分别做什么？"

✅ 应清晰答出三层职责。

### 测试 2：知识库连通

> 让 Agent："帮我看一下知识库 TPR 目录下有哪些项目"

✅ 应能调用知识库 API 返回结果。

### 测试 3：文件上传

> 发送一个文件给 Agent，观察是否确认路径后上传。

✅ 应展示文件清单、确认路径、上传后展示云端结果。

---

## 常见问题

**Q：不配置知识库能用吗？**
A：能。TPR 核心功能（思维模式、GRV、Battle）完全不依赖知识库。

**Q：多个 Agent 能共享同一个知识库目录吗？**
A：能。用同一个 `kb_root_folder` 配置即可。建议按 `kb_project_id` 分空间隔离。

**Q：appKey 怎么获取？**
A：联系知识库服务团队。

**Q：如何关闭知识库同步？**
A：将 `kb_sync` 设为 `false` 或删除 `tpr_config` 配置块即可。
