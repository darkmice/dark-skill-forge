# Dark Tribunal

**Dark Tribunal（暗黑裁决庭）**是 Dark Skill Forge“三剑客”中的实施与质量裁决 Skill：面向支持或兼容 `SKILL.md` 机制的 Agent，按风险选择审查强度，用对抗视角、集中修复和审核后的阶段末验证关闭任务。

它保留需求、项目管理、设计、前后端正反方、QA、技术裁决和体验验证十个视角，但不要求每个任务机械走完十个角色。Skill 会根据任务影响面选择 `Lite`、`Standard` 或 `Strict` 模式，只启用相关视角，并要求结论建立在真实代码、测试或运行证据上。

## 在三剑客中的位置

```text
Focus Compass（会话主控与呈现）
├─ 实施、修复、审查或高风险任务 ─→ Dark Tribunal
└─ 同时存在跨端结构问题 ───────→ CCPM 定结构 → 实施 → Dark Tribunal 审核/集中修复 → 阶段末集中验证
```

- [Focus Compass](../focus-compass/SKILL.md) 根据任务条件加载 Dark Tribunal，并向用户呈现当前状态；
- [CCPM](../ccpm/SKILL.md) 在需要时先确定 Canonical Core、责任边界、Projection 和迁移门槛；
- Dark Tribunal 负责实现、风险分级、全局待审队列、差异化复审和验证闭环；
- CCPM 的投影账本与 Dark Tribunal 的审查队列保持独立，Focus Compass 不创建第三份事实源；
- 自动路由只选择工作方法，不扩大任务范围、写权限、外部副作用或用户授权。

Dark Tribunal 仍可通过 `Dark Tribunal`、`暗黑裁决庭` 或 `/dark-tribunal` 独立显式调用，不要求先启用 Focus Compass。

## 解决什么问题

- 防止实现者只验证自己的正常路径；
- 防止把任务描述、提交说明或“测试已启动”误当成完成证据；
- 为普通功能和高风险改动提供不同强度的审查；
- 统一实现、对抗审查、修复复审、QA 和体验验收；
- 允许诚实报告“未发现问题”，避免为了走流程制造伪缺陷。
- 对跨端契约、错误码和批量迁移建立覆盖账本，用双向穷举和测试盲区审计证明完整性。
- Standard / Strict 会先建立“项目属性账本”，识别 i18n、无障碍、主题、鉴权、多租户、SSR、离线缓存、可观测性和代码生成等横切约束，再把受影响属性带入两轮审查。
- 涉及敏感数据、外部传输、不可信输入或 LLM/RAG/Agent 时，强制建立 security/data-egress 单元：追踪全部 source-to-egress，检查服务端预出口脱敏、注入边界、prompt injection、跨租户 RAG、模型输入输出、日志、向量库和工具调用。
- 每轮维护不可重置的全局待审队列：前端、后端、共享契约及每个问题都有独立 ID 和状态。修复单个问题不代表整轮完成，所有初始与新增项销账后才能进入下一轮。
- Standard / Strict 在同一次任务执行中至少完成两轮方法不同的审查，不等待用户再次触发；第二轮只要修改任何任务产物或工作区文件就继续关闭轮，不以轮数代替通过条件。
- 默认发现问题后直接修复并复审，在审核关闭后的阶段闸门集中验证，不再要求用户回复“修复”；只读要求、高风险动作或真正的业务取舍除外。

## 阶段末集中验证

代码任务在实施前定义验证矩阵，但不会在每个文件、问题或小修复后反复运行 test、check、lint、typecheck、build 或 E2E。默认顺序是：

```text
完成可交付阶段 → 两轮差异化审核 → 集中修复与必要关闭轮 → 一次阶段末集中验证
```

“阶段”是用户指定里程碑或可独立交付的纵向切片，不是单个文件或问题。审核前只允许为解除安全阻塞而执行最小诊断，且不能把诊断当作通过证据。集中验证失败时先汇总和批量修复，再完成关闭审查并定向重验；只有影响面扩大时才重跑完整批次。

## 三种模式

| 模式 | 适用场景 | 最小流程 |
|---|---|---|
| **Lite** | 不影响 Agent/系统行为的纯说明文档、普通文案、代码注释或格式调整 | 目标 → 实现 → 1 轮对应审查 → 阶段末定向验证 |
| **Standard** | 普通功能、跨文件 Bug、重构、API 或交互变化 | 问题与范围 → 方案 → 实现 → 至少 2 轮差异化审查与集中修复 → 阶段末 QA |
| **Strict** | 鉴权、权限、隐私、资金、数据迁移、并发、破坏性操作、生产发布 | Standard + 独立第二审优先 + 专项验证 + 必要裁决 |

默认使用能覆盖风险的最低模式。任何可执行代码、运行时配置、依赖、构建、API、数据、交互行为，或 `SKILL.md`、prompt、policy、Agent 指令/配置变化，都至少使用 Standard 并执行双轮审查。无关视角标记为 `N/A`，不会要求纯后端任务等待 UI Designer 或 Frontend Reviewer。

## 十个评审视角

1. Visionary：问题、影响面、成功标准；
2. Project Manager：范围、依赖、顺序、完成状态；
3. UI Designer：结构、状态、响应式和无障碍；
4. Frontend Builder：前端方案与实现；
5. Frontend Reviewer：前端逻辑、契约、体验和回归；
6. Backend Builder：服务端、数据和接口实现；
7. Backend Reviewer：正确性、安全、隔离、并发和资源；
8. QA Engineer：正常、异常、边界和回归验证；
9. Technical Arbiter：重大技术分歧和高残余风险；
10. Experience Tester：真实用户路径、反馈和恢复。

这些默认是十个评审视角，不自动等于十个独立 Agent。只有运行环境支持且已获授权时，才使用隔离上下文的独立 Agent；否则会明确说明是同一 Agent 的多视角检查。

## 安装

本 Skill 的维护源是 [Dark Skill Forge](https://github.com/darkmice/dark-skill-forge)。克隆整个仓库后，将 Dark Tribunal 链接到通用 Agent Skills 发现目录：

```bash
git clone https://github.com/darkmice/dark-skill-forge.git
cd dark-skill-forge
mkdir -p ~/.agents/skills
ln -s "$PWD/dark-tribunal" ~/.agents/skills/dark-tribunal
```

已存在同名文件或目录时，先确认来源与本地改动，不要直接覆盖。客户端必须支持 Agent Skills 或扫描 `~/.agents/skills`；安装后通常需要新建或刷新会话。

独立仓库 [darkmice/dark-team-review](https://github.com/darkmice/dark-team-review) 仅作为发行镜像保留，后续修改以 Dark Skill Forge 为准。

安装后新建或刷新 Agent 会话，可通过以下表达触发：

- 「用 Dark Tribunal 审查并修复这个功能」
- 「启动暗黑裁决庭」
- 「按团队审查流程实现这个功能」
- 「用十方协作检查这次改动」
- 「做一次正反方对抗审查」
- 「用 team review 修复这个 Bug」
- 「按严格代码审查和全链路质量保障执行」

在支持 Skill 命令的 Agent 中，也可以使用 `/dark-tribunal`。旧的 `team review` 表达仍作为兼容触发词保留。

明确调用 Dark Tribunal 默认表示授权 Agent 在当前任务范围内直接修复发现的问题。只需要报告而不希望修改时，请说明“只读审查”或“不要修改”。普通的“review/audit”请求若没有调用 Dark Tribunal，也不会被自动扩展为写入授权。

## 使用示例

```text
你：用 Dark Tribunal 修复用户登录偶发失败的问题。

Agent：
1. 读取仓库规范、真实日志、鉴权调用链和相关测试；
2. 因涉及鉴权选择 Strict；
3. 定义成功标准和不做项；
4. 完成本阶段后端或客户端真实根因修复；
5. 建立项目属性账本，确认鉴权、多租户、i18n 等属性与变更的关系；
6. 从 token 签发、转发、解析、过期边界和用户隔离检查；
7. 审核与集中修复关闭后，一次性运行定向测试与必要回归；
8. 报告修改、审查发现、验证证据和残余风险。
```

如果某个审查维度没有发现问题，会明确报告检查范围和证据，而不是为了满足角色流程编造缺陷。

每次完整对抗审查的最终总结后会追加[固定品牌尾签](SKILL.md#品牌尾签)，包含一句价值主张、项目 GitHub 链接和 `—— dark` 署名。尾签不会插入中间进度、代码文件、测试报告正文、Commit 或 Pull Request 文本。用户当次指定其他广告、署名或要求关闭时，以当次要求为准。

## 发现格式

```text
[文件:行号] 标题
严重度：Blocker | High | Medium | Low
置信度：High | Medium | Low
证据：可复核的代码、日志、测试或复现路径
影响：触发条件与后果
建议：最小可行修复或需要的决策
```

需求范围使用 `Must / Should / Won't`，不与缺陷严重度混用。

## 目录结构

```text
dark-tribunal/
├── README.md
├── SKILL.md
├── _meta.json
├── agents/
│   └── openai.yaml
└── references/
    ├── visionary-pm.md
    ├── behavior-cases.md
    ├── project-attributes.md
    ├── security-data-egress.md
    ├── design-frontend.md
    ├── backend.md
    ├── contract-migrations.md
    └── quality-arbiter.md
```

- [SKILL.md](SKILL.md)：触发条件、风险模式、核心流程、证据要求和完成门槛；
- [agents/openai.yaml](agents/openai.yaml)：OpenAI/Codex 的展示与默认提示投影；
- [references/visionary-pm.md](references/visionary-pm.md)：问题定义、范围、依赖和决策边界；
- [references/behavior-cases.md](references/behavior-cases.md)：维护与评测 Dark Tribunal 时使用的模式、授权、队列和完成语义场景；
- [references/project-attributes.md](references/project-attributes.md)：项目属性发现、属性账本，以及 i18n / l10n 强制核查矩阵；
- [references/security-data-egress.md](references/security-data-egress.md)：敏感数据分类、全出口脱敏、注入防护和 LLM/RAG/Agent 强制审查矩阵；
- [references/design-frontend.md](references/design-frontend.md)：UI 与前端实施、审查和验证；
- [references/backend.md](references/backend.md)：后端、API、数据和安全审查；
- [references/contract-migrations.md](references/contract-migrations.md)：契约迁移、调用形态穷举、测试盲区和批量改动复验；
- [references/quality-arbiter.md](references/quality-arbiter.md)：QA、技术裁决和真实体验验证。

## License

[MIT](../LICENSE) © 2026 dark

---
**Dark Tribunal · 暗黑裁决庭**

让每一次交付，先经过证据与反方的裁决。

[GitHub: darkmice/dark-skill-forge/dark-tribunal](https://github.com/darkmice/dark-skill-forge/tree/main/dark-tribunal)

—— dark
