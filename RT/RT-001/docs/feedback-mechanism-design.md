# TPR 框架"零门槛"反馈机制设计方案

> **状态**：📋 待实现（Backlog）
> **归属**：RT-001 / TPR Framework
> **创建日期**：2026-04-08
> **仓库**：https://github.com/evan-zhang/tpr-framework

---

## 背景

用户在使用 TPR Skill 时可能需要反馈 Bug、提需求或改进建议，但并非所有用户都有 GitHub 账号。需要一套"零门槛"的反馈通道，让 Agent 代替用户完成 Issue 提交。

## 推荐方案：远程代理提交

**核心思路**：用户和 Agent 永远不接触 Token，由后端服务代为提交。

```
用户表达反馈意图
    │
    ▼
Agent 整理标题 + 内容
    │
    ▼
脚本调用远程 API：POST /api/tpr/feedback
body: { "title": "...", "body": "...", "labels": ["user-feedback"] }
    │
    ▼
远程服务用内部保管的 Token 提交 GitHub Issue
    │
    ▼
返回 Issue URL 给用户 ✅
```

### 为什么不直接发 Token 给用户？

| 风险 | 说明 |
|------|------|
| 多 IP 触发风控 | 同一 Token 从全球不同 IP 使用，GitHub 可能自动封禁 |
| 单点故障 | Token 被撤销后所有用户同时失效 |
| 共享被滥用 | 拿到 Token 的人可刷垃圾 Issue |

### 代理方案优势

- Token 零泄露（用户侧无凭证）
- 可加反垃圾（服务端限频，如每人每天 3 条）
- Token 可热更换（服务端换 Token，用户端零感知）
- 开箱即用（无需用户配置任何东西）

## 待工程师实现的清单

1. **部署一个极简 API 服务**（建议 Cloudflare Workers / Vercel Serverless）
   - 端点：`POST /api/tpr/feedback`
   - 入参：`{ "title": string, "body": string }`
   - 内部调用 GitHub API 创建 Issue
   - 可选：基础频率限制

2. **在 Skill 中新增脚本** `scripts/submit-feedback.sh`
   - 调用上述 API
   - Agent 在 SKILL.md 中获得触发指令

3. **更新 SKILL.md**
   - 新增"💡 用户反馈通道"章节
   - 当用户说"反馈/Bug/建议"时，Agent 自动整理内容并调用脚本

4. **备选过渡方案**（服务未就绪时）
   - 引导有 GitHub 账号的用户自行前往 Issues 页面提交
   - 地址：https://github.com/evan-zhang/tpr-framework/issues
