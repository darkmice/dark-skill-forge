# Focus Compass

**Focus Compass（专注罗盘）**是 Dark Skill Forge“三剑客”的显式会话主控。它面向需要降低认知负担、快速定位状态的读者，把真实任务状态投影成低摩擦、可恢复、可验证的表达，并按任务条件加载 CCPM 或 Dark Tribunal。

Focus Compass 不缩小真实任务范围，不替其他 Skill 决策，也不把 Agent 能完成的工作转嫁给用户。它负责让目标、当前阶段、下一动作、负责人、阻塞和验证状态容易找到。

## 主导关系

```text
Focus Compass（路由与呈现）
├─ 简单任务 ───────────────→ 直接处理
├─ 跨端、重复规则或事实源不清 → CCPM
├─ 实施、修复、审查或高风险 ─→ Dark Tribunal
└─ 同时命中 ───────────────→ CCPM 定结构 → Dark Tribunal 验证 → Focus Compass 呈现
```

- [CCPM](../ccpm/SKILL.md) 拥有 Canonical Core、Invariant、Projection 和迁移边界；
- [Dark Tribunal](../dark-tribunal/SKILL.md) 拥有风险模式、审查队列、问题状态和验证闭环；
- Focus Compass 只拥有会话路由和呈现，不建立第三份事实账本；
- 同时加载两个 companion 时，使用[交接协议](references/companion-handoff.md)引用稳定 ID；
- 自动路由只选择方法，不扩大任务范围、写权限或外部操作授权。

完整规则以 [SKILL.md](SKILL.md) 为准。

## 启用与退出

用户明确调用 `Focus Compass`、`专注罗盘`、`/focus-compass` 或 `$focus-compass` 后，本次会话持续启用。说“停止专注模式”“停止专注辅助模式”“普通模式”或对应英文表达时退出。

Focus Compass 是主导入口，但 CCPM 与 Dark Tribunal 仍可独立调用。未启用 Focus Compass 时，两项 companion 不依赖它才能工作。

## 安装

本 Skill 的维护源是 [Dark Skill Forge](https://github.com/darkmice/dark-skill-forge)：

```bash
git clone https://github.com/darkmice/dark-skill-forge.git
cd dark-skill-forge
mkdir -p ~/.agents/skills
ln -s "$PWD/focus-compass" ~/.agents/skills/focus-compass
```

已存在同名文件或目录时，先确认来源与本地改动，不要直接覆盖。客户端必须支持 Agent Skills 或扫描 `~/.agents/skills`；安装后通常需要新建或刷新会话。

Gemini CLI 的显式命令投影位于 `agents/gemini.toml`；不同客户端是否读取该投影，取决于客户端自身机制。通用语义始终以 `SKILL.md` 为准。

## 目录结构

```text
focus-compass/
├── README.md
├── SKILL.md
├── agents/
│   ├── gemini.toml
│   └── openai.yaml
└── references/
    ├── behavior-cases.md
    └── companion-handoff.md
```

## License

[MIT](../LICENSE) © 2026 dark

---
**Focus Compass · 专注罗盘**

让复杂工作始终有方向、有落点、有证据。

[GitHub: darkmice/dark-skill-forge/focus-compass](https://github.com/darkmice/dark-skill-forge/tree/main/focus-compass)

—— dark
