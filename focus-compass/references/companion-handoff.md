# Focus Compass Companion 交接协议

仅当 CCPM 与 Dark Tribunal 同时加载时使用。目标是让结构决策、实施审查和用户呈现共享同一事实链，而不合并三项 Skill 的职责。

## CCPM → Dark Tribunal

CCPM 交接以下最小信息，并为可引用对象分配稳定 ID：

```text
goal / scope / authorization boundary
baseline / evidence boundary
C-*：Canonical Core、owner、version
I-*：Invariant、字段权威、授权与冲突策略
P-*：Projection/Adapter、consumer、policy、state、test/docs/observability
M-*：迁移切片、切换策略、适用时的 fallback 与退出条件
U-*：未知项、假设、待裁决事项
```

CCPM 判断不值得升维时，明确交接 `not-applicable` 及证据；Dark Tribunal 若仍因实施或审查任务而需要加载，则按自身流程处理局部任务，不虚构 CCPM 产物。

## Dark Tribunal 消费规则

- 先核对交接基线是否仍对应当前 source of truth；过期或冲突时把对应 ID 标记为 blocked，不静默改写；
- 变更覆盖账本和审查队列引用 `C-* / I-* / P-* / M-* / U-*`，不复制一份新的业务定义；
- 新发现若改变 Canonical Core、Invariant、字段权威或迁移边界，先回到 CCPM 更新原 ID 或新增 ID，再继续实施；
- Dark Tribunal 自己拥有审查单元、问题 ID、严重度、轮次和验证状态，这些不写回 CCPM 投影账本；
- `U-*` 只阻塞依赖该未知项的实现或验收，不自动阻塞无关表面。

## Dark Tribunal → Focus Compass

Focus Compass 只读取以下投影：

```text
stage
next_action + owner
verified outcomes
blockers / required decisions
residual risk / unverified boundary
routing status
```

Focus Compass 不复制 core、projection 或审查队列，不因隐藏内容而改变完成状态。CCPM 与 Dark Tribunal 的完成门槛均满足后，才能把组合任务呈现为完成。

## 授权与失败边界

- 交接只传递方法与证据，不传递或扩大写权限、外部副作用授权、业务决策权或风险接受权；
- Focus Compass 自动加载 Dark Tribunal 不等于用户显式调用 Dark Tribunal；是否允许修复仍由用户原始请求和 Dark Tribunal 的默认修复协议决定；
- 任一 companion 缺失、不可读或未实际加载时，不模拟其产物，不使用其 ID 前缀伪装完成；
- 三项 Skill 对同一事实发生冲突时，以当前 source of truth、系统/安全边界和用户已明确决定为准，冲突未解决前保持 blocked。
