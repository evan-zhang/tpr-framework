# 安装后配置

如果你是编排型 agent 且需要跑 TPR 全流程，建议在 AGENTS.md 中声明：

| 声明项 | 说明 | 示例 |
|--------|------|------|
| tpr_mode | 使用模式 | cognitive / full |
| can_spawn | 是否能派生 sub-agent | true / false |
| rt_root_dir | 本地 RT 项目目录的根路径 | ~/.openclaw/.../workspace/projects |

> 详细的安装指引见项目根目录 [INSTALL.md](../INSTALL.md)。
