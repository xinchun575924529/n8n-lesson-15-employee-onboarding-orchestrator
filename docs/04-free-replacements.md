# 04 · 凭证与端点替代方案

## 结论：本模板零替代需求

四个系统的 HTTP 端点默认全部指向 **httpbin.org**（公共免费回显服务）：

| 系统 | 端点 | 凭证 |
|------|------|------|
| HR | `https://httpbin.org/anything/hr/employees` | 无 |
| IAM | `https://httpbin.org/anything/iam/users` | 无 |
| IT | `https://httpbin.org/anything/it/tickets` | 无 |
| SaaS | `https://httpbin.org/anything/saas/apps` | 无 |

无 LLM 节点 → 无 token 费用。httpbin 对 POST 原样回显 body/headers，足够验证编排逻辑。

## 要接真实系统时怎么换

| 目标 | 换法 |
|------|------|
| 真实 HR（如 BambooHR） | 改 URL + 加 Header Auth 凭证；hrId 提取路径按真实响应改 |
| 内网系统 | httpbin 换内网 mock（如自建 json-server / webhook.site） |
| 离线教学 | 用 L2-B 的 mock 工作流（4 个 Code 节点顶替 HTTP），完全离线可跑 |
| 故障演练 | 保留 `/status/500` 改道机制，或指向真实系统的测试环境 |

## httpbin 不可用时

- 自建：`docker run -p 8080:80 kennethreitz/httpbin`
- 轻量替代：webhook.site（在线）/ n8n 自家 Webhook 测试 URL 回环