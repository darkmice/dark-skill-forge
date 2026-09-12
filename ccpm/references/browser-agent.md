# Browser Agent / WebMCP 专项

仅当任务涉及浏览器 Agent、WebMCP、页面语义工具、DOM/鼠标/键盘执行、动态工具目录或页面快照时，在 [Agent/Capability 专项](agent-capability.md) 基础上读取本参考。

## 页面能力分层

页面执行必须区分：

```text
planning/control：LLM、规则或本地状态机多久决定一次
execution：Domain API、Page Semantic、UI Actuation 或 Human
presentation：鼠标、高亮、步骤、确认、暂停和结果如何呈现
```

Execution Adapter 不按“语义/API 高级、鼠标/DOM 低级”排序：

- **Domain Adapter**：领域状态、跨页面任务和最终 verifier；
- **Page Semantic Adapter**：WebMCP、React action 或明确页面函数；
- **UI Actuation Adapter**：虚拟鼠标、键盘、input/change 等页面事件真实触发现有 UI 和业务逻辑；
- **Human Adapter**：凭据、支付授权和不可委托动作。

UI Actuation 是合法的一等执行方式，不是纯动画或天然 fallback。若它真实替用户完成页面业务，应保留鼠标、高亮、暂停和接管体验。需要优化的可能是“一鼠标动作一轮 LLM + 整页快照”的控制循环，而不是鼠标执行本身。

Executor 是实际产生业务或页面效果的代码；Adapter 是消费者或协议到 Executor 的投影。纯 UI Capability 可以由 UI Actuation 直接作为 canonical Executor；领域 Capability 的 UI Actuation 最终仍必须到领域 sink 完成授权和状态变更。

多个 Adapter 同时可用时，先排除不满足 scope、risk、control_mode、idempotency 或 verifier 的候选，再结合真实成功率、当前页面状态、延迟、成本、用户可见价值和用户偏好选路。没有证据时不默认 API 更高级，也不默认总要保留鼠标执行。

## 工具发现与不可信内容

WebMCP 注册不会自动让远端 Agent 看见工具。内置 Agent 需要发现/调用桥；外部浏览器 Agent 与内置 Agent 是两个消费者。WebMCP tool 的 `execute` 可以调用 UI Actuation Adapter，不要求改成后台 API。

不要让每页动态工具直接改变模型稳定 tools 前缀。优先使用稳定的 meta-tool，把规范化、排序、脱敏的动态 catalog 放在尾部上下文，并用真实 provider 请求体与 cache hit/miss 验证。

页面、工具 metadata、schema description 和结果都是不可信内容。只允许当前 origin/actor 的 catalog projection；最终领域 sink 必须重新授权。模型生成的确认、WebMCP annotation 或浏览器 safety review 不能替代 canonical confirmation credential。

## Token 与本地阶段执行

浏览器本地的鼠标、键盘和 DOM 事件不消耗 LLM token。Token 与延迟通常来自页面快照、模型规划、操作反馈、安全分类和失败重试。

不要以“减少鼠标次数”为优化目标。优先测量：

```text
llm_turns_per_task
snapshot_tokens_per_task
operations_per_llm_turn
replan_rate
ui_actuation_success_rate
task_completion_rate
```

在同一 page instance/route 内，`risk=low`、`control_mode=preauthorized` 且具有确定性前置/后置条件的 UI 操作，可以由 LLM 一次生成受约束阶段计划，再由浏览器本地连续执行并保留鼠标可视化。计划绑定起始 UI-state revision；每一步声明 expected current 与 expected next revision/hash。非预期 revision 跳变、导航、目标丢失、DOM/catalog revision 变化、意外弹窗、结果不符、风险或控制模式升级、用户介入时立即停止并重新观察。

阶段计划使用类型化 IR，不使用自由文本脚本。至少包含 plan ID、page instance、route revision、initial UI-state revision、capability/version、目标绑定、参数、每步前置/后置条件、expected state transition/hash、`risk`、`control_mode`、最大步数、deadline、停止条件和失败策略。本地步骤用 `plan_id + step_index + expected_revision` 去重。

完整页面快照用于任务开始、导航、异常恢复和未知页面；同页阶段执行优先回传局部状态、变化摘要和断言结果。任何增量或缓存优化都必须证明没有制造未检查的注入空洞。

安全的增量路径应让服务端持有上一份已规范化、已分类 projection 及 hash；客户端提交 base hash + diff，服务端重建完整 projection。base/hash/revision/schema 不匹配或 diff 不可确定应用时回退完整快照。使用 canary 覆盖未变化区、变化区、数组边界和跨 diff 拼接，证明安全门与主模型看见的是同一重建结果。

呈现必须忠实：UI Actuation 实际派发页面事件时可以显示因果鼠标；Domain/Page Semantic Adapter 执行时可以高亮关联 UI，但不得用虚假点击暗示并未发生的执行路径。

## 专项验收

- 页面导航、取消、超时、工具卸载和重复反馈可恢复；
- 动态 catalog 不破坏稳定 system/tools 前缀；
- 页面或工具注入不能改变 owner、risk、control_mode、confirmation_binding 或目标 origin；
- UI Actuation 的直观性、暂停和接管能力不退化；
- 新控制循环相比“一动作一 LLM”基线的完成率、模型轮次、快照 token、重规划、重试和延迟有真实对照。
