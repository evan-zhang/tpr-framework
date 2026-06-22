# tpr-framework Discussion Log

## 2026-04-01 — 质检驱动重构

**背景**：xgjk-skill-auditor 审计 D1 得 4/10（189行超标），D3 大量 NEVER 无解释，D5 design/ 缺失。

**决策**：
- Bindings Management + Sub-agent Spawning 两章推入 references/
- Critical Rules 补"为什么"说明
- 补建 design/ 档案
- 发布到 ClawHub（tpr-framework）

**产出**：SKILL.md 189→57行，references/ 新建 spawning-guide.md + bindings-guide.md

## 2026-06-20 — v3.1.0 知识库同步回归

**背景**：v3.0.0 移除了知识库同步（依赖 openclaw-xgkb-sync 独立服务），但实际使用中 Agent 产出的文件需要自动同步到知识库。用户希望零进程依赖、实时同步。

**讨论要点**：
1. 方案对比：独立同步服务 vs fire-on-write 内嵌同步 → 选 fire-on-write
2. 配置方式：每个项目一份 .xgkb.json vs 项目集合根目录统一一份 → 选统一配置
3. 二进制文件支持：uploadContent 仅支持文本，需要 uploadWholeFile + saveFileByPath(nameConflictStrategy=1)
4. 幂等性验证：2026-06-21 验证通过，nameConflictStrategy=1 对 PDF 幂等有效

**决策**：
- 新建 xgkb-sync-helper 独立 Skill（https://github.com/evan-zhang/xgkb-sync-helper）
- TPR 通过 orchestrator-ops.md 规则引用 xgkb-push
- projects/.xgkb.json 统一配置
- 文本走 uploadContent，二进制走 uploadWholeFile + saveFileByPath

**产出**：v3.1.0 发布，包含 SKILL.md 知识库同步章节、INSTALL.md 安装步骤、orchestrator-ops.md v2.5.0 同步规则
