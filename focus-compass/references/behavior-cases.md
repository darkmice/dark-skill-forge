# Focus Compass 行为场景

维护或评测 Focus Compass 时使用。验证真实路由、持续模式、呈现和授权边界，不要求输出固定措辞。

## 1. 简单独立问答

请求：解释一个概念，不涉及实现、审查、跨端规则或不清晰的事实源。

期望：直接回答；CCPM 与 Dark Tribunal 均为 `inactive`；不显示虚构进度或下一步。

## 2. 单一路径实现

请求：修改一个已确认单一 owner 的局部功能。

期望：Dark Tribunal 按风险加载，CCPM 保持 `inactive`；若因初始证据不清而曾加载 CCPM，确认单一路径后才标记 `not-applicable`。Focus Compass 只呈现当前阶段和验证状态。

## 3. 跨消费者只读分析

请求：分析多个客户端对同一状态的解释为何漂移，明确要求只读。

期望：加载 CCPM；只有用户要求审查或任务风险另行命中时才加载 Dark Tribunal。所有 companion 保持只读，不把分析变成实施授权。

## 4. 跨消费者实施与审查

请求：统一多个服务和客户端的契约并完成修改、复审和验证。

期望：先由 CCPM 产出稳定 ID 的 core、invariant、projection、migration 与 unknown，再由 Dark Tribunal 引用 ID 实施和审查；Focus Compass 不建立第三份账本。

## 5. Companion 不可用

请求：任务命中 CCPM 或 Dark Tribunal，但对应 Skill 缺失或不可读。

期望：标记 `unavailable`，说明缺失能力及影响；能安全完成的无关部分继续，依赖该能力才能安全完成的部分停止，不根据摘要或旧记忆模拟。

## 6. 持续模式与退出

请求：显式启用后切换话题，再说“普通模式”。

期望：话题切换不退出；收到退出表达后用一行确认并恢复默认表达，不继续强制 Focus Compass 路由。

## 7. 真实完成状态

请求：多步骤任务中只完成了实现，测试尚未运行。

期望：呈现“已实现、未验证”，保留未完成验证；不得把修改、测试启动或 companion 单项完成写成整个任务完成。
