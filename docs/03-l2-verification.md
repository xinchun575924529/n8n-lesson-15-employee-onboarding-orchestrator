# 03 · L2 三关验证记录

## A 关 · 单元级（56/56 通过）

忠实移植 6 个 Code 节点进独立测试工作流（`L2Test205840001`），逐单元断言：

| 单元 | 断言数 | 覆盖 |
|------|-------|------|
| U1 校验规整 | 20 | 必填、邮箱正则、别名、默认值、simulateStatus 改道 |
| U2 HR 结果处理 | 9 | 2xx 判定、hrId 提取/回退、string body 容错 |
| U3 IAM 结果处理 | 9 | userId 三来源、失败回退、双 payload 组装 |
| U4 IT 结果处理 | 5 | ticketId、404、透传 |
| U5 终态汇总 | 6 | completed/partial/failed 三态 |
| U6 响应组装 | 7 | 201/207/502 状态码映射 |

## B 关 · mock 全链（6 场景 39/39 通过）

保留模板全节点连线，4 个 HTTP 节点换 mock Code：

| 场景 | 结果 |
|------|------|
| S1 全成功 | 201 / completed / 4 OK |
| S2 HR 挂 | 207 / partial / 3 OK，hr.ok=false 正确记账 |
| S3 缺 firstName | valid=false 不短路，仍 201（缺陷实锤②） |
| S4 非法邮箱 | 同上（缺陷实锤②） |
| S5 IAM+SaaS 双挂 | 207 / partial / 2 OK |
| E1 全灭 | 502 / failed / 0 OK |

## C 关 · 真跑（httpbin 真链路，3/3 符合预期）

模板原版导入本机 n8n 激活，curl 真打 `http://127.0.0.1:5678/webhook/employee-onboard`：

| 发次 | 剧情 | 实测 |
|------|------|------|
| run1 | 正常全流程 | **HTTP 201**，completed，4/4 OK，8.3s |
| run2 | simulateStatus=500（HR 故障注入） | **HTTP 207**，partial，3/4 OK，hr.ok=false status=500，9.4s |
| run3 | 缺 firstName（缺陷复现） | **HTTP 201** completed（valid 未短路，缺陷实锤③），9.2s |

零费用（httpbin 免费），无 LLM。