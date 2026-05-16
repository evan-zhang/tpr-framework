# 知识库同步

> sync_service_url 需在 AGENTS.md 中声明，详见 references/setup.md

TPR 通过 `openclaw-xgkb-sync` 同步服务将产出自动同步到玄关知识库。

## 同步模式：全局 Mapping

在 `openclaw-xgkb-sync` 的 `config.json` 中注册一条覆盖整个 `rt_root_dir` 的 mapping，所有 TPR 项目自动同步。

## 工作机制

1. **编排者写文件** — TPR 编排者或 sub-agent 将产出写入 `{rt_root_dir}/{项目编号}/` 下的本地目录
2. **服务自动同步** — `openclaw-xgkb-sync` 每 120 秒扫描本地目录，自动将变更推送到知识库（bidirectional，LWW）
3. **编排者不参与任何同步操作**

## 查询真实同步状态

TPR 可以随时调用以下接口获取真实状态：

**服务健康检查**：
```bash
curl http://{sync_service_url}/health
```

**查看当前所有 mapping**：
```bash
curl http://{sync_service_url}/mappings
```

**触发立即同步**（非强制）：
```bash
curl -X POST http://{sync_service_url}/sync
```

## 编排者职责

- **只写本地文件**，不参与任何同步逻辑
- 同步服务不可用时不阻塞主流程，继续写本地文件即可
