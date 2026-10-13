# Cloudflare 定时调度

`cronjob` 提供 Cronhub 页面、管理 API 和 `SchedulerAlarm` Durable Object，使用现有 D1 数据库和 Secrets。

首次 API 请求通过内部绑定启动唯一的 `cronhub` Alarm 实例。Cloudflare 保存下一分钟的唤醒时间，到期后执行 Alarm；每次 Alarm 在扫描任务前保存下一次唤醒，因此无需浏览器保持打开或外部服务持续请求。

`SchedulerAlarm` 使用 SQLite 存储，可在 Cloudflare Workers Free 计划使用。它只保存调度唤醒时间，GitHub 账号与任务继续保存在原 D1 数据库中，无需增加 Secrets。

任务执行时间、时区、启用状态和下次执行时间都在 Cronhub 管理。早报任务使用 `Asia/Shanghai` 时区和 `30 8 * * *`，对应每天北京时间 08:30。

GitHub Actions 中的早报 Workflow 只配置 `workflow_dispatch`，由 Cronhub 在任务到期时触发。

## 部署

仓库的 Deploy Workflow 自动部署 Worker 和 Alarm 类，并保留现有 `TOKEN_ENC_KEY`。本地部署命令：

```powershell
npm ci
npm run typecheck
npm run build
npx wrangler deploy
```

首次部署后登录 Cronhub，API 请求会自动启动 Alarm。后续部署继续使用已有实例和已保存的唤醒时间。保留 `wrangler.jsonc` 的 `durable_objects` 和 `exports` 配置。

## 验证

Cronhub 仪表盘的调度器心跳应每分钟更新；关闭浏览器后心跳仍应继续更新。运行记录中的 `schedule` 表示任务由定时调度触发。GitHub Actions 的最终运行结果和钉钉响应共同确认早报发送结果。
