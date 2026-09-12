# CKAP：Agent/Capability 专项

当任务涉及 Agent、LLM、RAG、Tool、Skill、MCP、WebMCP、浏览器自动化或模型副作用时，在 CCPM 主流程上使用本专项。

## 专项定义

CKAP（Capability Kernel–Adapter Projection）是 CCPM 在 Agent 系统中的 profile：

> 业务能力、权限、风险、确认、执行、审计和结果验证形成 Capability Kernel；Human UI、Agent Tool、WebMCP、MCP/API、自动化和人工客服是 Adapter Projection。

Tool、Skill 和 prompt 都不是业务事实源：

- Tool 是 Capability 的模型投影；
- Skill 是能力选择与组合说明；
- prompt 约束模型行为，但不授权、不确认、不执行；
- WebMCP/MCP 是协议适配器，不拥有业务规则。

## Capability Contract 下限

```text
canonical ID + version
owner
input/output/error schema
actor / tenant / resource scope
side effect / reversibility / idempotency
risk: low | high | manual
confirmation mode and binding
data classification and model egress policy
executor
result verifier
audit fields
adapter projections
```

`preview / confirm / execute / verify` 通常是同一 Capability invocation 的生命周期，不应随意拆成互不关联的业务能力。

## 责任边界

- 领域服务拥有业务状态、执行器和最终授权；
- Agent 负责编排和提出动作，不拥有权限；
- Gateway/Auth 提供已验证身份，不替领域服务推断 owner；
- Adapter 只投影，不复制状态机；
- Data Policy 在模型、日志、Memory、tool result 等出口前执行；
- verifier 决定业务目标是否真正达成，HTTP 2xx/tool success 不自动等于完成。

## 页面 Agent 与 WebMCP

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

UI Actuation 是合法的一等执行方式，不是纯动画或天然 fallback。若它真实替用户完成页面业务，应保留其鼠标、高亮、暂停和接管体验。需要优化的常常是“一鼠标动作一轮 LLM + 整页快照”的控制循环，而不是鼠标执行本身。

术语边界：Executor 是实际产生业务/页面效果的代码；Adapter 是消费者或协议到 Executor 的投影。纯 UI Capability 可以由 UI Actuation 直接作为 canonical Executor；领域 Capability 的 UI Actuation 最终仍必须到领域 sink 完成授权和状态变更。

多个 Adapter 同时可用时，先排除不满足 scope、risk、confirmation、idempotency 或 verifier 的候选，再结合真实成功率、当前页面状态、延迟/成本、用户可见价值和用户偏好选路。没有证据时不默认 API 更高级，也不默认总要保留鼠标执行。

WebMCP 注册不会自动让远端 Agent 看见工具。内置 Agent 需要发现/调用桥；外部浏览器 Agent 与内置 Agent 是两个消费者。WebMCP tool 的 `execute` 可以调用 UI Actuation Adapter，不要求改成后台 API。

不要把每页动态工具直接改变模型稳定 tools 前缀。优先使用稳定的 meta-tool，把规范化、排序、脱敏的动态 catalog 放在尾部上下文，并用真实 provider 请求体与 cache hit/miss 验证。

页面工具 metadata、schema description 和结果是不可信内容。只允许当前 origin/actor 的 catalog projection；最终领域 sink 再授权。

## Token 与本地阶段执行

浏览器本地的鼠标、键盘和 DOM 事件不消耗 LLM token。Token/延迟通常来自页面快照、模型规划、操作反馈、安全分类和失败重试。

不要以“减少鼠标次数”为优化目标。优先测量：

```text
llm_turns_per_task
snapshot_tokens_per_task
operations_per_llm_turn
replan_rate
ui_actuation_success_rate
task_completion_rate
```

在同一 page instance/route 内，`low` 风险且具有确定性前置/后置条件的 UI 操作可以由 LLM 一次生成受约束阶段计划，再由浏览器本地连续执行并保留鼠标可视化。计划绑定起始 UI-state revision；每一步声明 expected current 与 expected next revision/hash，执行成功后按预期转移推进。用户、异步请求或其他代码造成的非预期 revision 跳变，以及导航、目标丢失、DOM/catalog revision 变化、意外弹窗、结果不符、`high/manual` 或用户介入，都会立即停止并重新观察。

阶段计划使用类型化 IR，不使用自由文本脚本。至少包含 plan ID、page instance、route revision、initial UI-state revision、capability/version、目标绑定、参数、每步前置/后置条件、expected state transition/hash、risk、最大步数、deadline、停止条件和失败策略。`low` 来自 canonical policy；“可确定执行”要求每步效果可本地断言，且失败/重试不会产生未识别的重复副作用。React 内部状态变化不能只靠 URL 感知，应推进显式 UI-state revision；本地步骤用 `plan_id + step_index + expected_revision` 去重。

完整页面快照用于任务开始、导航、异常恢复和未知页面；同页阶段执行优先回传局部状态、变化摘要和断言结果。任何增量/缓存优化都必须证明没有制造未检查的注入空洞。

安全的增量路径应让服务端持有上一份已规范化、已分类 projection 及 hash；客户端提交 base hash + diff，服务端重建完整 projection。base/hash/revision/schema 不匹配或 diff 不可确定应用时回退完整快照。使用 canary 覆盖未变化区、变化区、数组边界和跨 diff 拼接，证明安全门与主模型看见的是同一重建结果。

呈现必须忠实：UI Actuation 实际派发页面事件时可以显示因果鼠标；Domain/Page Semantic Adapter 执行时可以高亮关联 UI，但不得用虚假点击暗示并未发生的执行路径。

## 副作用与确认

- `low`：授权后可自动执行并验证；
- `high`：服务端 preview，用户在可信 UI 确认，兑换绑定 actor/interaction/capability/version/args hash/expiry 的一次性凭证，再执行和验证；
- `manual`：密码、OTP、私钥、支付授权、提现等最终步骤只由当前最终用户完成，默认不可委托。

模型生成的 `confirm:true`、自然语言“用户已同意”、WebMCP annotation 或浏览器 safety review 都不是 Koda/产品自身的确认凭证。

## Agent 数据与注入边界

- 当前用户有权且任务必需的业务值可以进入模型，不能为安全把 Agent 变成瞎子；
- credentials、Cookie、Token、私钥和一次性 capability 永不进入模型；
- 用户输入、页面、RAG、Memory、工具 metadata/result 和模型输出全部不可信；
- 间接页面/工具注入应拦截或降级，但不能无证据归罪用户；
- 模型工具参数在执行前做 schema、scope、owner 和风险验证。

## 迁移切片

第一批选择低风险、可验证、已有多入口的能力。一个切片同时包含：

```text
canonical definition
domain/page executor
authorization and data policy
Agent projection
Human UI projection
optional WebMCP/MCP projection
audit and verifier
cross-adapter tests
fallback and rollout metrics
```

不要先批量包装按钮或 API，再把权限、确认和审计留到以后。

## 专项验收

- A 用户通过任意 Adapter 不能访问 B 用户资源；
- metadata/result prompt injection 不能改变 owner/risk/confirmation；
- 不支持协议、tool unregister、导航、取消、超时和重复反馈可恢复；
- high-risk 不能跨 interaction/capability/version/args 重放；
- tool success 但 verifier 未通过时报告 unknown/failed，而非完成；
- 动态 catalog 不破坏稳定 system/tools 前缀；
- UI Actuation 的鼠标直观性、暂停和接管能力不退化；
- 新控制循环相比“一动作一 LLM”基线的完成率、模型轮次、快照 token、重规划、重试和延迟有真实对照。
