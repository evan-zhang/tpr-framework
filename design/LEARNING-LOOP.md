# tpr-framework 学习复盘

## 2026-04-01

### Problem → Rule
**问题**：Critical Rules 用 NEVER/ALWAYS/Critical 大写，但未解释原因，AI 遵守规则的可靠性低于理解原因。
**规则**：每条强制规则必须附带"为什么"，让 AI 理解背后的逻辑，而非只能记忆死规则。

### Problem → Rule
**问题**：操作手册类内容（Bindings/Spawning）混入框架规则 SKILL.md，导致文件臃肿，触发时加载大量无关内容。
**规则**：SKILL.md 只放框架规则和角色边界。操作手册推入 references/，按需加载。

## 2026-06-20

### Problem → Rule
**问题**：v3.0.0 移除知识库同步后，Agent 产出文件只写本地磁盘，无法公网预览。用户反馈实际需要同步能力。
**规则**：移除一个能力前，确认是否有替代方案。如果没有，应该重构而非移除。

### Problem → Rule
**问题**：xgkb-sync-helper 的 SKILL.md 在迁移 commit 中被误删（commit message 说保留但实际删除），导致 Agent 无法发现该 Skill。
**规则**：commit message 与实际操作必须一致。迁移类 commit 后必须验证文件完整性。

### Improvement Direction
1. SKILL.md 当前 295 行，超出 SOP 建议的 200 行，考虑将「知识库同步」「项目与安装」等章节推入 references/
2. 版本号曾出现 _meta.json / CHANGELOG / SKILL.md 三处不一致，应增加 VERSION 文件作为唯一真相源
