---
name: ccpm
description: 使用 CCPM（规范内核—多投影法）把跨页面、协议、角色或消费者的重复问题收敛为统一事实内核、责任边界和可验证投影。适用于用户明确要求 CCPM、高维度/系统性解决、不要打补丁、统一事实源，或局部问题反复暴露跨端契约缺口的架构、重构和复杂修复；不用于事实证明是单一 owner、单一路径且无需共享抽象的普通局部修改。
license: MIT
---

# CCPM：规范内核—多投影法

CCPM（Canonical Core–Projection Method）把“高维度解决低维度问题”转成可执行工程方法：先找到应成为唯一权威的契约、状态机、数据模型或规则内核，再向 UI、API、任务、Agent、缓存、通知等消费者生成受控投影。

目标不是把问题说大，而是减少重复事实、重复判断和跨端漂移，使底层实现更简单、更一致、更可验证。

## 适用边界

满足任一情况时使用完整 CCPM：

- 同一业务语义存在于多个页面、服务、协议、角色或渠道；
- 修复一个入口后，同类问题会在其他入口复现；
- prompt、关键词、白名单、映射表或兼容分支持续增长；
- source of truth、owner、授权者、执行者或 verifier 不清晰；
- 新协议/新客户端可能复制现有业务和安全规则；
- 用户明确要求系统性、结构化、可扩展或“不要打补丁”。

以下情况不要强行升维：

- 根因已确认局限在一个 owner 和一个执行路径；
- 没有第二消费者、共享契约或跨端一致性需求；
- 抽象不会删除重复判断，只会增加间接层。

对这类任务直接做最小正确修复并验证。CCPM 不是“所有问题都新建平台或中心服务”。

## 核心术语

- **Canonical Core**：真正的权威内核，可以是领域契约、数据模型、状态机、规则、Policy、事件或设计系统，不必是单一服务。
- **Projection**：内核面向某个消费者的表示或接口，例如 UI、DTO、API、Agent Tool、缓存、通知或报表。
- **Adapter**：完成协议/运行环境转换的实现。Adapter 不拥有业务语义。
- **Invariant**：无论入口、协议、语言、角色和环境如何变化都必须成立的规则。
- **Fallback**：迁移或能力不足时可选的兼容路径；一旦使用，必须有范围、指标和退出条件。

不要按技术表象给 Adapter 排高低。API、页面动作、任务、人工流程或批处理只要真实承载执行并服从同一内核，就可以是一等 Projection。应分别审查“谁作决定”“怎样执行”和“如何呈现”，不能把其中一层的问题误判成另一层的错误。

## 工作流

### 1. 先判断是否值得升维

读取当前事实源、调用链、消费者、配置、测试定义/历史结果和文档；此处是建立事实与验证计划，不提前执行常规测试或检查。区分：

- 用户目标；
- 表面症状；
- 已确认事实；
- 合理推测；
- 未知或必须由用户决定的取舍。

如果不同假设会导致不同内核，不要先写抽象；列出决定性证据和待裁决项。

### 2. 建立投影账本

至少登记：

```text
source / canonical definition
transform / mapper / generator
projection / adapter / sink
consumer
policy / authorization / data boundary
state / persistence / cache
test / docs / observability
```

从 source 正向追到消费者，再从消费者反向追到 source。发现同一规则有多份定义时，先标出实际 authority，不要凭命名判断。

### 3. 选择 Canonical Core

候选内核必须同时满足：

1. 靠近真实业务 owner；
2. 能表达稳定语义，而不是当前页面或协议细节；
3. 可以生成或校验多个投影；
4. 能承载版本、权限、状态和验证边界；
5. 引入后能删除重复决策，而不是增加一层同步。

若没有候选满足这些条件，报告“尚不能确定统一内核”，不要虚构中央平台。

### 4. 定义不变量和字段权威

逐项拍板：

- 谁定义 canonical ID、schema 或状态机；
- 谁认证 actor，谁在最终 sink 授权；
- 谁定义风险、确认、幂等和数据策略；
- 谁执行，谁验证结果，谁负责审计；
- 冲突时是覆盖、取交集还是构建失败。

默认规则：各职责权威约束取交集，任一安全/权限边界拒绝即拒绝；Projection/Adapter 只能收窄，不能扩大。

### 5. 设计 Projection

每个 Projection 只做消费者需要的转换：

- 明确输入输出与 canonical identity；
- 不复制领域状态机、价格计算、owner 推断或授权规则；
- 动态可用性只能收窄；
- 不兼容变化有版本和迁移；
- projection 集合能与 source 自动或人工对账。

同时把三个维度分开：

```text
decision / planning：谁决定目标和下一阶段
execution：通过 API、服务、任务、界面或人工怎样真正执行
presentation：用户如何看见、理解、暂停和接管
```

优化其中一层，不得无证据删除另外两层已有价值。先证明瓶颈属于决策、执行还是呈现，再改变对应机制。

若任务涉及 Agent、LLM、MCP、WebMCP、Tool 或 Skill，必须读取 [Agent/Capability 专项](references/agent-capability.md)。

若任务进一步涉及浏览器 Agent、WebMCP、页面语义工具、DOM/鼠标/键盘执行、动态工具目录或页面快照，必须同时读取 [Browser Agent 专项](references/browser-agent.md)。普通 LLM、RAG、后端 Agent 或非页面 Tool 任务不要加载该专项。

若需要选择不同领域的内核形态，读取 [常见领域模式](references/domain-patterns.md)。

### 6. 规划迁移，不默认瞬时替换

迁移单位是完整纵向切片：core、policy、executor、projection、consumer、test、docs 和观测一起闭环。

- 先冻结事实基线和覆盖账本；
- 在实施前定义每个纵向切片的阶段完成条件与验证矩阵；单个文件、问题或小修复不能临时当成阶段；
- 根据可逆性、兼容性和数据一致性选择版本并存、feature detection/flag、原子切换或其他策略；
- 只有旧路径不会破坏一致性或安全边界时才把它作为 fallback，并记录使用指标；
- 新路径真实 E2E 后才删除重复；
- 使用了 fallback 却没有退出条件时，不得称为迁移完成。

### 7. 用证据关闭任务

CCPM 负责定义要验证的不变量、Projection、消费者和失败场景，不拥有组合模式下测试与检查的执行调度。若任务进入实施，完整读取并加载 [Dark Tribunal](../dark-tribunal/SKILL.md)：先完成纵向切片实现与 Dark Tribunal 审核/集中修复，再由 Dark Tribunal 在阶段闸门统一运行 test、check、lint、typecheck、build、E2E 或真实操作；不要在每个 Projection 或小修复后重复执行。若任务仅为分析或架构设计，不运行实现验证并明确边界；若实施所需的 Dark Tribunal 缺失或不可读，只交付 CCPM 结构与验证矩阵，并把实施闭环标记为 `unavailable`，不得模拟其审核或宣称阶段完成。

验证至少覆盖：

- 正常路径；
- 权限、owner/tenant、空值和错误类型；
- 每个 Projection 的同语义对账；
- 旧客户端、降级、重试、并发和部分失败；
- 反例、其他支持语言和真实 E2E；
- “调用成功但目标状态未达成”的 verifier 场景。

测试通过只证明测试观察到的范围。生产、外部系统或真实用户链未验证时明确披露。

## 与 Dark Tribunal 的阶段验证交接

CCPM 向 Dark Tribunal 交接独立的 `V-*` 验证记录；存在迁移时，每个 `V-*` 关联对应的 `M-*` 纵向切片：

```text
V-* / related M-* / stage boundary / completion condition
validation matrix：invariant / projection / consumer / scenario / expected evidence
validation status：planned / deferred-until-review / running / passed / failed / blocked
```

- `planned` 只表示验证已设计，不表示运行或通过；
- Dark Tribunal 接管实施/审查后，CCPM 不再启动同一批验证；
- 审核期间发现新的消费者或不变量时，更新原验证矩阵，仍留到阶段末集中执行；
- 集中验证失败若暴露内核、字段权威或迁移边界错误，先更新对应 `C-* / I-* / P-* / M-*`，再交回 Dark Tribunal 批量修复、关闭审查与定向重验；
- 用户明确要求 TDD、项目硬性门禁或安全 Blocker 的例外，由 Dark Tribunal 记录原因并控制最小执行范围。

## 交付形态

根据用户请求选择最小必要产物：

- **分析/诊断**：输出症状、真实目标、投影账本、内核候选、决定性证据和未知项；保持只读。
- **架构/RFC**：使用 [CCPM 决策记录模板](references/decision-record.md)，拍板内核、不变量、责任和迁移门槛。
- **实现/重构**：先确认目标内核、授权范围、纵向切片和验证矩阵，再交由 Dark Tribunal 完成实施、审核与阶段末集中验证。
- **普通局部修复**：若第 1 步证明不需要共享内核，直接修复，不制造 CCPM 仪式。

维护或评测 CCPM 本身时，使用 [行为场景](references/behavior-cases.md) 检查应进入、应退出、信息不足、授权边界和专项路由，不能只依赖 frontmatter 或链接校验。

## 质量门槛

完成时应能明确回答：

1. 真正问题是什么，哪些只是症状？
2. Canonical Core 是什么，为什么它有权威？
3. 每个字段和不变量由谁负责？
4. 有哪些 Projection/Adapter，是否存在第二事实源？
5. Adapter 是否扩大了权限或复制业务规则？
6. 是否把决策频率、执行机制和用户呈现错误地绑定在一起？
7. 如使用 fallback，其范围、指标和退出条件是什么？
8. 哪些证据证明跨投影一致，哪些仍未验证？
9. 验证矩阵是否只由 Dark Tribunal 在审核关闭后集中执行，是否避免按文件、Projection 或小修复重复运行？

答不出来时不要用“高维方案”“平台化”“统一架构”等抽象词宣称完成。
