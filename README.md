# Lesson 15 · 员工入职编排引擎（Webhook 跨 HR / IAM / IT / SaaS 四系统打通）

> 源模板：[n8n.io/workflows/20584](https://n8n.io/workflows/20584/) · 20 节点 · **零凭证开箱即跑**

一条 POST 请求触发整条入职流水线：**外部系统（OA/招聘）→ Webhook → 字段校验 → 依次开通 HR 档案、IAM 账号、IT 工单、SaaS 应用 → 聚合结果按 completed / partial / failed 三态动态回 201 / 207 / 502**。

## 为什么值得学

- **企业级编排范式**：串行 Saga 式调用链 + 单点失败不炸全链（neverError + continueOnFail 三件套）
- **动态 HTTP 状态码**：200 一把梭的接口很常见，会按业务结果回 201/207/502 的才是生产级
- **故障注入教学**：调用方传 `simulateStatus: 500` 即可让任意环节打到 httpbin 故障端点，演示「部分失败」不用改代码
- **零成本**：四个系统端点全部指向 httpbin.org 公共回显服务，无 API key、无 LLM 费用

## 本课实锤的原版缺陷（教案级卖点）

**校验失败不短路**：`valid: false` 时流程照样把四个系统全调一遍再回 201。L2 三条路线（A 单元 / B mock / C 真跑）三次实锤。修复方案见 `workflow-custom.json`（加固版）。

## 文件导览

| 文件 | 说明 |
|------|------|
| `workflow.json` | 原版模板（可直导入 n8n 真跑） |
| `workflow-custom.json` | 加固版：校验短路回 422，修复上述缺陷 |
| `docs/00-overview.md` | 课程总览与学完后能做什么 |
| `docs/01-architecture.md` | 20 节点逐段拆解 |
| `docs/02-pitfalls.md` | 实测坑与排障 |
| `docs/03-l2-verification.md` | L2 三关验证记录（95 断言全绿 + 3 次真跑） |
| `docs/04-free-replacements.md` | 凭证/端点替代方案 |
| `docs/05-hardening.md` | 生产化加固清单 |
| `exercises/exercise.md` | 课后练习 |
| `script/short-video.md` | 获客短视频脚本 |

## 快速开始

1. 导入 `workflow.json` 到 n8n 并激活
2. `curl -X POST http://<host>:5678/webhook/employee-onboard -H "Content-Type: application/json" -d @payload.json`
3. 把 `simulateStatus` 改成 `500` 再发一次，观察 207 partial