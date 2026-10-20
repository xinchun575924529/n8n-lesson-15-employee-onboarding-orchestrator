# 课后练习

## 练习 1（基础）：跑通并复现缺陷
1. 导入 `workflow.json` 激活
2. 正常 payload 打一发 → 确认 201
3. 删掉 `firstName` 再打一发 → 观察「缺字段仍 201」的缺陷行为

## 练习 2（核心）：修复缺陷
导入 `workflow-custom.json`，重复练习 1 第 3 步 → 应回 **422** 且 body 带 issues 明细。
对比两个工作流的节点差异，说出改了哪三处。

## 练习 3（进阶）：故障注入
正常 payload 加 `"simulateStatus": 500` → 应回 207，body 里 hr.ok=false。
思考：如果要让「只有 SaaS 段」故障，改哪个节点的哪行代码？

## 练习 4（挑战）：并行化改造
把 IAM / IT / SaaS 三段改成并行分支（Split Out / 直接分支 + Merge），
保持三态汇总逻辑不变，全链时长应从 ~9s 降到 ~4s。

## 练习 5（挑战）：补偿机制
给每段加一个「撤销」mock 节点，当 overallStatus=failed 时逆向调用已成功段的撤销，
输出补偿执行报告。