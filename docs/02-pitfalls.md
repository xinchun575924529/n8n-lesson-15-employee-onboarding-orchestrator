# 02 · 实测坑与排障

## 坑 1：校验失败不短路（原版缺陷，三次实锤）

请求缺 `firstName` 或邮箱非法时，「收集校验结果」正确输出 `valid:false`，
但流程**照样调完四个系统并回 201 completed**——节点图上根本没有 If 分支。

- L2-A U1 单元断言实锤
- L2-B S3/S4 全链 mock 实锤
- L2-C run3 真跑实锤（缺 firstName，httpbin 真链路照回 201，8.3s 全链）

**修复**：见 `workflow-custom.json`——「收集校验结果」后插 If 节点，false 分支直接回 422 + issues 明细。

## 坑 2：PowerShell 直接 curl 内联 JSON 必坏

`curl -d "{"a":1}"` 在 PowerShell 里引号被吃掉，n8n 回 422 `Failed to parse request body`。
**一律用文件**：`curl.exe -X POST ... -H "Content-Type: application/json" --data @body.json`。

## 坑 3：CLI 导入工作流必须补 id

`n8n import:workflow` 对无 `id` 字段的 JSON 报
`SQLITE_CONSTRAINT: NOT NULL constraint failed: workflow_entity.id`。
导入前先写入唯一 id（如 `L2Cstm20584001`）。

## 坑 4：webhook 改动后要重启才生效

`update:workflow --active` / `publish:workflow` 之后，生产 webhook 路由由常驻进程持有，
**重启 n8n 进程**后新激活的工作流才开始接流量。激活后立刻 curl 会 404。

## 坑 5：旧版 CLI 弃用警告

`n8n update:workflow --active=true` 已弃用（仍能跑），新写法 `n8n publish:workflow --id <id>`。

## 坑 6：Task Broker 端口冲突

本机常驻 n8n 已占 5689，CLI 起临时实例要设 `N8N_RUNNERS_BROKER_PORT` 到空闲端口（如 5691），否则启动报错。

## 坑 7：CLI 激活后旧进程内存路由不刷新（2026-10-17 实锤）

publish:workflow 改的是 DB（active + activeVersionId），但**正在运行的 n8n 进程内存里的 webhook 路由表不会自动重载**——表现为 workflow_entity/webhook_entity 全部正确、webhook 照样 404 Cannot POST。
解法：**真重启 n8n 进程**（确认新进程 CreationDate 晚于 publish 时间；本机有 watchdog 会代劳但时机不可控）。
验证加固版时先 unpublish 同路径的旧工作流，避免两条流抢一个 webhook path。