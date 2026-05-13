# 产出交付协议

> 本文档只回答一个问题：**TPR 产出文件怎么交付给用户。**

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
TPR/{项目编号}/
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
└── 05-closure/
    └── P-ACPT-01.md
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
| 鉴权 | Header `appKey` |
| 核心接口 | `POST /document-database/file/uploadContent` |

### 新建文件

```bash
curl -X POST 'https://sg-al-cwork-web.mediportal.com.cn/open-api/document-database/file/uploadContent' \
  -H 'appKey: {APP_KEY}' \
  -H 'Content-Type: application/json' \
  -d '{
    "content": "{MD 或 HTML 内容}",
    "fileName": "DISCOVERY",
    "fileSuffix": "md",
    "folderName": "TPR/TPR-20260513-001/01-discovery"
  }'
```

### 更新文件版本（核心机制）

同一个文件被修改时（如 Battle 后 GRV 修订），使用 `updateFileId` 覆盖更新：

```bash
curl -X POST 'https://sg-al-cwork-web.mediportal.com.cn/open-api/document-database/file/uploadContent' \
  -H 'appKey: {APP_KEY}' \
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

## 错误处理

| 场景 | 处理 |
|------|------|
| uploadContent 返回 resultCode ≠ 1 | 重试 1 次，仍失败则报告错误，不阻塞流程 |
| fileId 映射表丢失 | 按 folderName + fileName 查询知识库，恢复 fileId |
| HTML 生成失败 | 只交付 MD 版本，告知用户 HTML 版本待补充 |
| appKey 无效 | 立即停止交付，报告用户 |
| 限流 (resultCode 610012) | 等待 2 秒重试，最多 3 次 |

---

*版本：2.2.0*
*创建：2026-05-13*
