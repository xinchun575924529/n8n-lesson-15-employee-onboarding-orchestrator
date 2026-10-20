# 00 · 课程总览

## 一句话

用 n8n 把一个「新员工入职」业务事件，编排成跨 HR / IAM / IT / SaaS 四个系统的自动化开通流水线，并返回生产级的三态结果。

## 业务背景

企业入职场景里，一个新人要在 N 个系统里开账号：HR 系统建档、IAM 开身份、IT 派工单（电脑/门禁）、SaaS 开应用（邮箱/IM/文档）。人工做是半天活，还漏。本模板是这类「跨系统编排（orchestration）」的最小完整范例。

## 学完你能

1. 用 Webhook + Code 节点搭一条串行编排链
2. 掌握「单点失败不炸全链」三件套：`neverError`（httpbin 不回 4xx/5xx 传输错误）+ `continueOnFail` + 每段结果独立记账
3. 按业务结果动态回 HTTP 状态码（201/207/502）
4. 用 `simulateStatus` 做故障注入演练
5. 识别并修复「校验不短路」这类模板级缺陷

## 前置

- 会导入/激活 n8n 工作流
- 会发 curl POST
- 不需要任何付费账号

## 节点构成（20 个）

16 功能节点：Webhook、Code × 6、HTTP Request × 4、Wait、Respond to Webhook、Set；4 张教学便签。