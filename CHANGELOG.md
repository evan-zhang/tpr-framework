# TPR Framework — Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

---

## [3.0.0] - 2026-05-19

### Changed
- **移除知识库同步相关内容** — TPR Skill 职责边界明确为纯方法论 + 本地产出规范
  - 删除 `references/setup-sync.md`（知识库同步配置）
  - 删除 `docs/INSTALL.md`（旧版安装文档）
  - 删除 `docs/kb-sync-design-discussion.md`（历史设计讨论）
  - 清理 README.md、INSTALL.md、SKILL.md、output-delivery.md、orchestrator-ops.md、setup.md 中的同步引用
  - 知识库同步由 `openclaw-xgkb-sync` 独立服务管理，TPR 不再涉及

---

## [2.2.0] - 2026-05-13

### Added
- **Consensus-Divergence Mapping** in Best-Minds protocol (`references/tpr-cognitive.md`)
  - Flexible range guidance: 3-5 consensus models, 2+ divergences
  - Self-check rules against false consensus and straw-man divergences
  - Mapping results flow into existing GRV fields (constraints + risks)
- **Domain cognitive completeness review** dimension in Battle (`references/battle-protocol.md`)
  - Auditor checks consensus/divergence coverage without forced counts

---

## [2.1.0] - 2026-04-08

### Added
- Best-Minds expert thinking protocol (`references/tpr-cognitive.md`)
  - Expert self-check checklist
  - Automatic domain identification mapping
  - Expert rigor in Battle reviews
- Expert identity injection in orchestrator dispatch (`references/orchestrator-ops.md`)
- Metrics, Self-Fix, and Knowledge Flywheel mechanisms
- Comprehensive acceptance tests and walkthrough guide in README

### Changed
- Migrated all legacy naming (三省六部制) to modern architecture naming (策划层/审查层/执行层)
- Restructured README installation guide for AI Agent direct consumption

---

## [2.0.0] - 2026-04-07

### Added
- Complete TPR Framework rewrite (Think / Probe / Review)
- Three-layer architecture: Planner / Auditor / Executor
- Four-stage execution: DISCOVERY → GRV → Battle → Implementation
- GRV contract format standard
- Battle mechanism with state machine
- Orchestrator operations manual
- Three project templates (Simple / Standard / Complex)
- Micro T/P/R protocol for Implementation phase
- Self-improvement system (corrections.md + patterns.md)
- Project grading system
