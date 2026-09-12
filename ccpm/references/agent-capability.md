# CKAP：Agent/Capability 专项

当任务涉及 Agent、LLM、RAG、Tool、Skill、MCP 或模型副作用时，在 CCPM 主流程上使用本专项。浏览器 Agent、WebMCP、页面语义工具、DOM/鼠标/键盘执行、动态工具目录或页面快照另行读取 [Browser Agent 专项](browser-agent.md)。

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
risk: low | high
control_mode: preauthorized | confirm-before-execute | manual-only
confirmation_binding
data classification and model egress policy
executor
result verifier
audit fields
adapter projections
```

`preview / confirm / execute / verify` 通常是同一 Capability invocation 的生命周期，不应随意拆成互不关联的业务能力。

`risk` 表示潜在影响，`control_mode` 表示谁在何时控制执行，两者不能混成同一枚举。高风险通常需要交互确认或人工完成，但最终组合由 canonical policy 决定，Adapter 只能收窄。

## 责任边界

- 领域服务拥有业务状态、执行器和最终授权；
- Agent 负责编排和提出动作，不拥有权限；
- Gateway/Auth 提供已验证身份，不替领域服务推断 owner；
- Adapter 只投影，不复制状态机；
- Data Policy 在模型、日志、Memory、tool result 等出口前执行；
- verifier 决定业务目标是否真正达成，HTTP 2xx/tool success 不自动等于完成。

## 副作用与确认

- `preauthorized`：仅在现有授权明确覆盖当前参数和副作用时自动执行并验证；
- `confirm-before-execute`：先生成 preview，用户在可信界面确认，再兑换绑定 actor/interaction/capability/version/args hash/expiry 的一次性凭证并执行；
- `manual-only`：密码、OTP、私钥、支付授权、提现等最终步骤只由当前最终用户完成，默认不可委托。

模型生成的 `confirm:true`、自然语言“用户已同意”或 Adapter metadata 都不是宿主产品或领域系统的确认凭证。

## Agent 数据与注入边界

- 当前用户有权且任务必需的业务值可以进入模型，不能为安全把 Agent 变成瞎子；
- 真实 credentials、Cookie、Token、私钥和一次性 capability 不得进入模型；安全测试使用不可兑换的合成 canary 或占位值；
- 用户输入、页面、RAG、Memory、工具 metadata/result 和模型输出全部不可信；
- 间接页面/工具注入应拦截或降级，但不能无证据归罪用户；
- 模型工具参数在执行前做 schema、scope、owner 和风险验证。

## 迁移切片

第一批选择低风险、可验证、已有多入口的能力。一个切片同时包含：

```text
canonical definition
canonical executor
authorization and data policy
Agent projection
Human UI projection
optional protocol/tool projections
audit and verifier
cross-adapter tests
fallback and rollout metrics
```

不要先批量包装按钮或 API，再把权限、确认和审计留到以后。

## 专项验收

- A 用户通过任意 Adapter 不能访问 B 用户资源；
- 不可信输入、RAG、Memory、tool metadata/result 和模型输出不能改变 owner、risk、control_mode、confirmation_binding 或授权边界；
- 不支持协议、tool unregister、取消、超时和重复反馈可恢复；
- high-risk 不能跨 interaction/capability/version/args 重放；
- tool success 但 verifier 未通过时报告 unknown/failed，而非完成；
- 新 Agent 路径相比现有路径的完成率、模型轮次、重试、延迟和成本有真实对照。
