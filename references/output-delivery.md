# 产出交付协议

> 本文档只回答一个问题：**TPR 产出文件怎么交付给用户。**

---

## 前置检查：是否启用知识库同步

在执行任何交付操作前，编排者必须先检查配置：

### Step 0：读取配置

从当前 Agent 的 AGENTS.md 中读取 `tpr_config` 配置块。

```yaml
# 示例配置（在 Agent 的 AGENTS.md 中声明）
tpr_config:
  kb_sync: true
  kb_appkey_env: KB_APP_KEY
  kb_root_folder: "TPR"
  kb_project_id: null
```

### Step 1：判定行为

```
检查 tpr_config 是否存在
 ├─ 不存在 → 本地模式（只写本地文件，不触知识库）
 │
 ├─ kb_sync = false 或未设置 → 本地模式
 │
 ├─ kb_sync = true
 │   ├─ kb_appkey_env 未设置 → 报错，降级为本地模式
 │   ├─ process.env[kb_appkey_env] 为空 → 报错，降级为本地模式
 │   └─ 配置合法 → 知识库模式（本地文件 + 知识库同步）
 │
 └─ 配置异常 → 报错，降级为本地模式
```

**降级原则**：任何配置问题都不阻塞 TPR 主流程。先完成本地文件，再报告配置错误。

---

## 核心原则

**双轨交付，统一存储**：

- **MD 文件** → 给 AI 读（RAG 索引、Agent 上下文）
- **HTML 文件** → 给人读（可视化预览）
- **统一存到知识库**，不做第二套存储

---

## 存储架构

### 知识库目录结构

```
{kb_root_folder}/{项目编号}/
├── 01-discovery/
│   ├── DISCOVERY.md           ← AI 消费
│   └── DISCOVERY.html         ← 人消费（关键文件才生成 HTML）
├── 02-planning/
│   ├── GRV.md
│   └── GRV.html
├── 03-battle/
│   ├── BATTLE-R1-AUDITOR.md   ← 只有 MD（过程记录，不需要 HTML）
│   └── BATTLE-R1-EXECUTOR.md
├── 04-execution/
│   ├── A001-OUT-01.md
│   └── A001-OUT-01.html       ← 最终交付物生成 HTML
├── 05-closure/
│   ├── P-ACPT-01.md
│   └── P-ACPT-01.html
└── kb-registry.yaml           ← 文件 ID 映射表（知识库版本管理核心）
```

### 哪些文件生成 HTML

| 文件 | 生成 HTML | 理由 |
|------|----------|------|
| DISCOVERY.md | ✅ | 用户需要审阅洞察报告 |
| GRV.md | ✅ | 用户需要审阅契约 |
| Battle 记录 | ❌ | 过程记录，只有 AI 需要看 |
| 执行日志 | ❌ | 过程记录 |
| 最终交付物 (A-OUT) | ✅ | 用户需要审阅成果 |
| 验收文档 (P-ACPT) | ✅ | 用户需要确认验收 |

---

## 知识库 API 使用

### 基础配置

| 配置项 | 值 |
|--------|-----|
| API 基地址 | `https://sg-al-cwork-web.mediportal.com.cn/open-api` |
| 鉴权 | Header `appKey`，值从 `process.env[tpr_config.kb_appkey_env]` 读取 |
| 核心接口 | `POST /document-database/file/uploadContent` |
| 目标空间 | `tpr_config.kb_project_id`（null = 个人知识库） |
| 根目录 | `tpr_config.kb_root_folder`（默认 "TPR"） |

### 新建文件

```bash
curl -X POST 'https://sg-al-cwork-web.mediportal.com.cn/open-api/document-database/file/uploadContent' \
  -H 'appKey: {从环境变量读取}' \
  -H 'Content-Type: application/json' \
  -d '{
    "content": "{MD 或 HTML 内容}",
    "fileName": "DISCOVERY",
    "fileSuffix": "md",
    "folderName": "{kb_root_folder}/{项目编号}/01-discovery",
    "projectId": "{kb_project_id，null则不传}"
  }'
```

### 更新文件版本（核心机制）

同一个文件被修改时（如 Battle 后 GRV 修订），使用 `updateFileId` 覆盖更新：

```bash
curl -X POST 'https://sg-al-cwork-web.mediportal.com.cn/open-api/document-database/file/uploadContent' \
  -H 'appKey: {从环境变量读取}' \
  -H 'Content-Type: application/json' \
  -d '{
    "content": "{更新后的内容}",
    "fileName": "GRV",
    "fileSuffix": "md",
    "updateFileId": {上次返回的 fileId},
    "versionName": "V2.0",
    "versionRemark": "Battle R1 修订：补充风险章节"
  }'
```

**关键**：`updateFileId` 不会改变文件的路径和 fileId，只是追加一个新版本。用户查看时永远看到最新版本，历史版本在知识库中保留。

---

## 多版本管理规范

### 版本编号规则

| 场景 | versionName | versionRemark |
|------|------------|---------------|
| 首次生成 | V1.0 | "初始版本" |
| Battle R1 修订 | V2.0 | "Battle R1 修订：[具体改动]" |
| Battle R2 修订 | V3.0 | "Battle R2 修订：[具体改动]" |
| 执行阶段微调 | V3.1 | "执行阶段更新：[具体改动]" |

### 文件 ID 映射表

编排者必须维护一个文件 ID 映射表，记录每个文件在知识库中的 `fileId`：

```yaml
# 文件位置：{项目目录}/kb-registry.yaml
project: TPR-20260513-001
kb_folder: "TPR/TPR-20260513-001"
files:
  discovery_md:
    fileId: 30001
    version: V1.0
    suffix: md
  discovery_html:
    fileId: 30002
    version: V1.0
    suffix: html
  grv_md:
    fileId: 30003
    version: V1.0
    suffix: md
  grv_html:
    fileId: 30004
    version: V1.0
    suffix: html
  battle_r1_auditor_md:
    fileId: 30005
    version: V1.0
    suffix: md
```

**用途**：
- 编排者每次需要更新文件时，从映射表查 `fileId`，用 `updateFileId` 参数调用 API
- 映射表本身也存到知识库（同目录下），确保跨 session 可恢复

### 版本更新流程

```
1. Sub-agent 完成文档 → 写入本地文件
2. 编排者读取 kb-registry.yaml
3. 检查该文件是否已有 fileId：
   ├─ 没有 → 新建（uploadContent，不传 updateFileId）
   │         → 拿到 fileId，写入映射表
   └─ 有 → 更新版本（uploadContent，传 updateFileId）
           → versionName 递增，versionRemark 写明原因
4. 如果该文件需要 HTML 版本：
   ├─ 用 doc-viewer 风格模板生成 HTML
   └─ 同样走 uploadContent（fileSuffix: html），独立 fileId
5. 更新 kb-registry.yaml 并同步到知识库
```

---

## HTML 生成规范

### 风格选择

参考 doc-viewer Skill 的风格体系，TPR 产出推荐：

| 文件类型 | 推荐风格 | 理由 |
|----------|---------|------|
| DISCOVERY（洞察报告） | 02-C（白底咨询报告） | 分析型内容，适合阅读 |
| GRV（项目契约） | 02-D（彩色模块矩阵） | 结构化内容，模块清晰 |
| 最终交付物 | 02-C 或按内容类型选择 | 视交付物性质而定 |
| 验收文档 | 01（综合风格） | 简洁确认型内容 |

### HTML 技术规范

- 单文件，内联 CSS
- 使用 TailwindCSS CDN
- 响应式设计
- 文件大小 < 1MB
- 知识库支持 HTML 直接预览

---

## 编排者操作检查清单

每个阶段完成时，编排者必须执行：

### DISCOVERY 完成后
- [ ] DISCOVERY.md → uploadContent (md)
- [ ] DISCOVERY.html → uploadContent (html, 风格 02-C)
- [ ] 更新 kb-registry.yaml

### GRV 完成后
- [ ] GRV.md → uploadContent (md)
- [ ] GRV.html → uploadContent (html, 风格 02-D)
- [ ] 更新 kb-registry.yaml

### Battle 每轮完成后
- [ ] BATTLE-R{n}-AUDITOR.md → uploadContent (md, 仅首次)
- [ ] BATTLE-R{n}-EXECUTOR.md → uploadContent (md, 仅首次)
- [ ] 如 GRV 被修订 → GRV.md + GRV.html 更新版本
- [ ] 更新 kb-registry.yaml

### Implementation 完成后
- [ ] 交付物.md → uploadContent (md)
- [ ] 交付物.html → uploadContent (html)
- [ ] 更新 kb-registry.yaml

### Closure 完成后
- [ ] P-ACPT.md → uploadContent (md)
- [ ] P-ACPT.html → uploadContent (html)
- [ ] 更新 kb-registry.yaml
- [ ] 通知用户：知识库目录路径 + 预览链接

---

## 用户文件上传管理

TPR 流程中用户经常提供参考文件（需求文档、数据表格、行业报告等）。这些文件应同步到知识库，与项目产出统一管理。

### 触发时机

- DISCOVERY 阶段用户提供参考材料
- 任何阶段用户主动发送文件要求上传
- 用户发送压缩包（自动识别并解压）

### 核心原则

**上传前必须确认路径。不自动瞎传。**

### 流程

```
收到用户文件（单文件 / 多文件 / 压缩包）
    │
    ├─→ 1. 文件识别
    │     ├─ 单文件：读取文件名、类型、大小
    │     └─ 压缩包（.zip/.tar.gz/.rar）：解压到临时目录，列出文件清单
    │
    ├─→ 2. 生成上传计划
    │     根据文件类型和内容推荐目标路径：
    │     ├─ 需求文档 (.md/.docx/.pdf) → {kb_root_folder}/{项目编号}/01-discovery/references/
    │     ├─ 数据文件 (.csv/.xlsx/.json) → {kb_root_folder}/{项目编号}/04-execution/data/
    │     ├─ 设计文档 (.md/.pdf) → {kb_root_folder}/{项目编号}/02-planning/references/
    │     └─ 其他 → {kb_root_folder}/{项目编号}/references/
    │
    ├─→ 3. 向用户展示并确认
    │     "以下文件将上传到知识库：
    │      📄 需求说明.md → TPR/TPR-20260513-001/01-discovery/references/
    │      📄 市场数据.csv → TPR/TPR-20260513-001/04-execution/data/
    │      📄 竞品分析.pdf → TPR/TPR-20260513-001/01-discovery/references/
    │      
    │      确认路径？[确认 / 修改路径 / 取消]"
    │
    ├─→ 4. 用户确认后执行
    │     ├─ 文本文件（.md/.txt/.csv/.json）→ uploadContent（纯文本通道）
    │     └─ 二进制文件（.pdf/.xlsx/.docx）→ 先上传到文件存储拿 resourceId，再绑定到知识库
    │
    └─→ 5. 返回结果
          每个文件的 fileId + 存储路径
          更新 kb-registry.yaml
```

### 压缩包处理

1. 解压到临时目录（`/tmp/tpr-upload-{timestamp}/`）
2. 递归列出所有文件（保留相对路径结构）
3. 按文件类型推荐目标路径（保留压缩包内的目录结构）
4. 向用户展示完整清单：
   ```
   📦 项目资料.zip（共 5 个文件）
   ├─ 📄 需求/PRD.md      → TPR/TPR-20260513-001/01-discovery/references/需求/
   ├─ 📄 需求/用户调研.md  → TPR/TPR-20260513-001/01-discovery/references/需求/
   ├─ 📊 数据/市场数据.csv → TPR/TPR-20260513-001/04-execution/data/
   ├─ 📊 数据/用户画像.xlsx → TPR/TPR-20260513-001/04-execution/data/
   └─ 📄 设计/UI规范.pdf   → TPR/TPR-20260513-001/02-planning/references/
   
   确认以上路径？[确认 / 修改 / 取消]
   ```
5. 用户确认后逐个上传
6. 完成后清理临时目录

### 上传方式选择

| 文件类型 | 上传方式 | 说明 |
|----------|---------|------|
| .md / .txt / .csv / .json | uploadContent（纯文本） | 直接读取内容，用纯文本通道上传 |
| .html / .htm | uploadContent（纯文本） | 同上 |
| .pdf / .docx / .xlsx / .pptx | uploadContent（纯文本）+ 附件 | 内容提纯后上传 md 版本，原文件作为附件说明 |
| .zip / .tar.gz / .rar | 先解压再上传 | 按压缩包流程处理 |
| 图片 / 其他 | uploadContent（纯文本） | 生成描述性 md 文件上传，标注原文件类型 |

### 红线

| # | 规则 |
|---|------|
| F1 | 上传前必须向用户展示完整文件清单 + 目标路径 |
| F2 | 必须等用户明确确认后才执行上传 |
| F3 | 用户可以修改任意文件的路径 |
| F4 | 超过 10 个文件时，按类型批量确认（不需要逐个确认） |
| F5 | 上传失败不阻塞流程，记录失败文件，最后报告 |
| F6 | 临时文件在流程结束后必须清理 |

---

## 知识库读取能力

TPR 不只是写入知识库，也需要从知识库中读取历史项目和相关文档。

### 读取场景

| 场景 | 时机 | 用途 |
|------|------|------|
| 浏览已有项目 | DISCOVERY 开始前 | 了解历史项目，避免重复工作 |
| 读取历史 GRV | 新项目策划时 | 参考类似项目的契约结构 |
| 读取项目文件 | 恢复中断的项目 | 跨 session 恢复上下文 |
| 搜索相关知识 | DISCOVERY 阶段 | 查找与当前项目相关的已有资料 |

### 接口使用

#### 1. 列出 TPR 目录下的项目

```bash
curl -s -X GET \
  'https://sg-al-cwork-web.mediportal.com.cn/open-api/document-database/file/searchFile?keyword=TPR&pageSize=50' \
  -H 'appKey: {从环境变量读取}'
```

或按目录浏览（需要先知道 TPR 目录的 fileId）：

```bash
curl -s -X GET \
  'https://sg-al-cwork-web.mediportal.com.cn/open-api/document-database/file/getChildFiles?parentId={TPR目录fileId}&pageSize=50' \
  -H 'appKey: {从环境变量读取}'
```

返回结果中每个项目是一个文件夹，包含 `fileId`、`fileName`、`fileType` 等信息。

#### 2. 列出项目内的文件

```bash
curl -s -X GET \
  'https://sg-al-cwork-web.mediportal.com.cn/open-api/document-database/file/getChildFiles?parentId={项目fileId}&pageSize=50' \
  -H 'appKey: {从环境变量读取}'
```

递归浏览直到找到目标文件。

#### 3. 读取文件全文（AI 首选）

```bash
curl -s -X GET \
  'https://sg-al-cwork-web.mediportal.com.cn/open-api/document-database/file/getFullFileContent?fileId={文件fileId}' \
  -H 'appKey: {从环境变量读取}'
```

返回提纯的 Markdown 全文，可直接用于 Agent 上下文。

#### 4. 批量读取多个文件

```bash
curl -s -X POST \
  'https://sg-al-cwork-web.mediportal.com.cn/open-api/document-database/ai/batchGetContent' \
  -H 'appKey: {从环境变量读取}' \
  -H 'Content-Type: application/json' \
  -d '{"files":[{"fileId":30001},{"fileId":30002}]}'
```

建议单次不超过 10 个文件。

#### 5. 搜索相关文档

```bash
curl -s -X GET \
  'https://sg-al-cwork-web.mediportal.com.cn/open-api/document-database/file/searchFile?keyword={关键词}&pageSize=20' \
  -H 'appKey: {从环境变量读取}'
```

### 读取检查清单

编排者在以下时点应主动读取知识库：

**DISCOVERY 开始前**：
- [ ] 列出 TPR 目录下的已有项目
- [ ] 搜索与当前需求相关的关键词
- [ ] 如有相关历史项目，读取其 GRV.md 作为参考

**恢复中断项目时**：
- [ ] 读取 kb-registry.yaml 获取 fileId 映射
- [ ] 读取最新阶段的文件，恢复上下文
- [ ] 通知用户："已从知识库恢复项目 {项目编号}"

**Battle 阶段**：
- [ ] 审查层可读取历史项目的 Battle 记录，参考常见异议点

---

## 错误处理

| 场景 | 处理 |
|------|------|
| uploadContent 返回 resultCode ≠ 1 | 重试 1 次，仍失败则报告错误，不阻塞流程 |
| 读取返回空内容 | 文件可能还在异步解析中，等待 3 秒重试 |
| fileId 映射表丢失 | 按 folderName + fileName 查询知识库，恢复 fileId |
| HTML 生成失败 | 只交付 MD 版本，告知用户 HTML 版本待补充 |
| appKey 无效 | 立即停止交付，报告用户 |
| 限流 (resultCode 610012) | 等待 2 秒重试，最多 3 次 |

---

*版本：2.2.0*
*创建：2026-05-13*
