# CCPM

**CCPM（Canonical Core–Projection Method，规范内核—多投影法）**是 Dark Skill Forge“三剑客”中的结构设计 Skill：把跨页面、服务、协议、角色或 Agent 的重复语义，收敛为唯一权威内核、清晰责任边界和可验证 Projection。

它解决的不是“怎样把局部问题说得更大”，而是判断问题是否真的跨越多个消费者；只有值得升维时，才从 Canonical Core 向 UI、API、任务、Agent、缓存、通知等出口生成受控投影。若事实证明问题只有单一 owner、单一路径且没有共享契约，CCPM 会回到最小正确修复。

## 在三剑客中的位置

```text
Focus Compass（会话主控与呈现）
└─ 跨端、重复规则、事实源或责任不清 ─→ CCPM（结构与迁移边界）
                                      └─ 需要实施或审查 ─→ Dark Tribunal
```

- [Focus Compass](../focus-compass/SKILL.md) 根据任务条件加载 CCPM，并向用户呈现当前状态；
- CCPM 负责 Canonical Core、Invariant、责任边界、Projection 和迁移门槛；
- [Dark Tribunal](../dark-tribunal/SKILL.md) 负责实现、风险分级、对抗审查和验证闭环；
- CCPM 的投影账本与 Dark Tribunal 的审查队列保持独立，Focus Compass 不创建第三份事实源；
- 自动路由只选择工作方法，不扩大任务范围、写权限、外部副作用或用户授权。

CCPM 仍可通过 `CCPM`、`规范内核—多投影法` 或 `/ccpm` 独立显式调用，不要求先启用 Focus Compass。

## 解决什么问题

- 同一业务语义散落在多个页面、服务、协议或角色中；
- 修复一个入口后，同类问题仍会在其他入口复现；
- prompt、关键词、白名单、映射表和兼容分支持续膨胀；
- source of truth、owner、授权者、执行者或 verifier 不清晰；
- 新客户端、新协议或新 Agent 重复复制业务与安全规则；
- 局部补丁暂时消除症状，却让系统继续漂移。

## 核心模型

- **Canonical Core**：权威契约、数据模型、状态机、规则、Policy、事件或设计系统；
- **Projection**：内核面向某个消费者的表示或接口；
- **Adapter**：只负责协议或运行环境转换，不拥有业务语义；
- **Invariant**：跨入口、协议、角色和环境都必须成立的规则；
- **Fallback**：迁移或能力不足时可选的兼容路径；一旦使用，必须有范围、指标和退出条件。

在 Agent 场景中，CCPM 使用 CKAP（Capability Kernel–Adapter Projection）专项：业务能力、权限、风险、确认、执行、审计和结果验证构成 Capability Kernel；Human UI、Agent Tool、WebMCP、MCP/API、自动化和人工客服是不同 Adapter Projection。Tool、Skill 与 prompt 都不是业务授权或事实源。

## 工作流

1. 判断问题是否真的值得升维；
2. 建立 source、transform、projection、consumer、policy、state、test、docs 和 observability 投影账本；
3. 选择靠近真实业务 owner、能表达稳定语义的 Canonical Core；
4. 定义字段权威、不变量、授权、执行、验证与冲突策略；
5. 为不同消费者设计只做必要转换的 Projection；
6. 以完整纵向切片迁移，按一致性与可逆性选择切换策略，并为实际使用的 fallback 定义退出条件；
7. 用正常路径、反例、权限边界、重试、并发、降级和真实 E2E 证据关闭任务。

完整规则以 [SKILL.md](SKILL.md) 为准。

## 安装

本 Skill 的维护源是 [Dark Skill Forge](https://github.com/darkmice/dark-skill-forge)。克隆整个仓库后，将 CCPM 链接到通用 Agent Skills 发现目录：

```bash
git clone https://github.com/darkmice/dark-skill-forge.git
cd dark-skill-forge
mkdir -p ~/.agents/skills
ln -s "$PWD/ccpm" ~/.agents/skills/ccpm
```

已存在同名文件或目录时，先确认来源与本地改动，不要直接覆盖。客户端必须支持 Agent Skills 或扫描 `~/.agents/skills`；安装后通常需要新建或刷新会话。

独立仓库 [darkmice/ccpm](https://github.com/darkmice/ccpm) 仅作为发行镜像保留，后续修改以 Dark Skill Forge 为准。

## 调用示例

- “用 CCPM 系统性解决这个问题”；
- “不要打补丁，找统一事实源”；
- “梳理 Canonical Core 和各端 Projection”；
- “从责任边界和不变量设计这次重构”；
- “检查是不是跨端契约漂移导致的重复问题”。

```text
你：用 CCPM 解决 Web、移动端和 Agent 对订单状态理解不一致的问题。

Agent：
1. 从三个消费者反查状态定义和真实写入链路；
2. 登记 source、mapper、projection、consumer、policy、test 和 docs；
3. 确认订单领域状态机是 Canonical Core；
4. 拍板状态 ID、迁移权限、事件版本和 verifier；
5. 让 Web、移动端和 Agent 只消费各自需要的受控 Projection；
6. 用新旧客户端、越权、重试、并发和跨投影对账验证迁移。
```

## 目录结构

```text
ccpm/
├── README.md
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── agent-capability.md
    ├── behavior-cases.md
    ├── browser-agent.md
    ├── decision-record.md
    └── domain-patterns.md
```

- [SKILL.md](SKILL.md)：适用边界、核心术语、完整工作流和质量门槛；
- [agents/openai.yaml](agents/openai.yaml)：OpenAI/Codex 的展示与默认提示投影；
- [references/agent-capability.md](references/agent-capability.md)：Agent、LLM、RAG、Tool、Skill 与 MCP 的通用 CKAP 专项；
- [references/browser-agent.md](references/browser-agent.md)：浏览器 Agent、WebMCP、页面执行、动态工具目录与快照专项；
- [references/behavior-cases.md](references/behavior-cases.md)：维护与评测 CCPM 时使用的进入、退出、授权和组合行为场景；
- [references/decision-record.md](references/decision-record.md)：CCPM 架构决策记录模板；
- [references/domain-patterns.md](references/domain-patterns.md)：常见领域的内核形态。

## License

[MIT](../LICENSE) © 2026 dark

---
**CCPM · 规范内核—多投影法**

让每一次复杂变更，都从唯一事实内核投影为一致交付。

[GitHub: darkmice/dark-skill-forge/ccpm](https://github.com/darkmice/dark-skill-forge/tree/main/ccpm)

—— dark
