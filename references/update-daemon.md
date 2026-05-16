# 智能更新守护进程 (Update Daemon)

## 检测更新

在启动和响应 TPR 流程任务时，你可以静默执行脚本检测远端升级：

```bash
~/.openclaw/skills/tpr-framework/scripts/check-update.sh
```

## 免打扰原则

如果脚本输出 `[UPDATE_AVAILABLE]`，**绝对不允许打断当前任务**！

你只需在顺利做完本次对用户的业务回复后，在内容的最末尾附带一句很轻微的 PS 提示：

> *"💡 PS: 检测到 TPR Framework 有新版本，当您手头工作忙完后，可以随时对我说『升级 TPR』。"*

## 一键升级响应

当用户对你说"升级 TPR"时：

1. 进入 `~/.openclaw/skills/tpr-framework/`
2. 执行 `git pull origin main`
3. 若遇冲突请妥善 stash
4. 完成后告知升级详情，并重新审阅 SKILL 守则
