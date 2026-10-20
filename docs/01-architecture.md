# 01 · 架构拆解

## 主链路（单线串联，无分支）

```
Webhook (POST /employee-onboard)
 → Code 收集校验结果（必填校验+邮箱正则+别名规整+simulateStatus 提取）
 → Code 整理员工数据（fullName/email/department/startDate/requestId 规整）
 → HTTP HR 系统建档（httpbin，simulateStatus 可改道 /status/500）
 → Code 处理HR（提取 hrId，成功/失败记账）
 → HTTP IAM 建账号（payload 带 hr 结果）
 → Code 处理IAM（userId 三来源提取）
 → HTTP IT 开工单
 → Code 处理IT（ticketId）
 → HTTP SaaS 开应用
 → Code 汇总状态（四系统 ok 计数 → completed/partial/failed）
 → Code 整理响应（组装响应体 + 状态码 201/207/502）
 → Wait（1s 缓冲）
 → Respond to Webhook
```

## 关键设计

### 1. 每段结果独立记账
每个系统调用后紧跟一个 Code 节点，输出统一结构：`{ ok, status, id, payload }`。
下游汇总节点只读这四个记账块，不关心系统细节——加第五个系统只需复制粘贴两段。

### 2. 失败不中断
HTTP 节点配置 `neverError`（配合 httpbin 的 `/status/500` 故障端点，500 也走正常输出）+ 工作流级 continueOnFail。
任一系统挂掉，其余三个照常执行，最后回 207 partial。

### 3. requestId 贯穿
入口生成/透传 requestId，每段 payload 携带，最终响应带回——排障定位的基本功。

### 4. simulateStatus 故障注入
请求体传 `simulateStatus: 500`，第一个 Code 节点把 HR（或可配其他段）的 URL 从 `/anything` 改道 `/status/500`。
教学演示「partial」场景零改造。

## 状态码映射

| 四系统结果 | overallStatus | HTTP |
|-----------|---------------|------|
| 4/4 ok | completed | 201 |
| 1~3 ok | partial | 207 |
| 0 ok | failed | 502 |