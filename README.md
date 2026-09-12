# Dark Skill Forge

**Dark Skill Forge（暗黑技能熔炉）**是 dark 的 Agent Skills 规范仓库，也是后续新 Skill 的统一事实源。

仓库首先收录一套可以组合使用的“三剑客”：Focus Compass 负责会话主控和注意力友好呈现，CCPM 负责复杂问题的规范内核与多投影设计，Dark Tribunal 负责风险分级、对抗审查和验证闭环。

## 三剑客

| Skill | 定位 | 触发边界 |
|---|---|---|
| [Focus Compass](focus-compass/README.md) | 会话入口、任务状态与低摩擦呈现 | 用户显式启用后持续生效，并按任务条件路由另外两个 Skill |
| [CCPM](ccpm/README.md) | 规范内核、责任边界、不变量与 Projection | 跨页面、服务、协议或角色存在重复语义和事实源漂移时使用 |
| [Dark Tribunal](dark-tribunal/README.md) | 实施、风险分级、双轮对抗审查与验证 | 功能、修复、重构、行为变化、审查及发布前验收时使用 |

组合顺序：

```text
Focus Compass（会话主控）
├─ 简单任务 ───────────────→ 直接执行并清晰呈现
├─ 跨端或事实源不清 ───────→ CCPM
├─ 实施、修复或审查 ───────→ Dark Tribunal
└─ 两者同时命中 ───────────→ CCPM 定结构 → Dark Tribunal 实施与验证 → Focus Compass 呈现
```

自动路由只选择工作方法，不扩大用户目标、任务范围、写入权限或外部操作授权。

三项同时工作时，CCPM 用稳定 ID 交接内核、不变量、Projection 与未知项，Dark Tribunal 引用这些 ID 建立审查覆盖，Focus Compass 只呈现状态。三者不复制业务事实，也不合并用途不同的账本。

## 仓库约定

- 每个 Skill 位于仓库根目录下的独立同名目录；
- `SKILL.md` 是该 Skill 的规范入口，条件性细节放入 `references/`；
- `agents/` 只存放不同 Agent 客户端的发现或调用投影；
- 本仓库是持续维护的 canonical source，已存在的独立仓库作为发行镜像保留；
- 新 Skill 必须完成结构校验、行为场景验证和差异化复审后，才可称为可用。

具体维护规则见 [AGENTS.md](AGENTS.md)。

## 全局安装

将仓库克隆到稳定目录：

```bash
git clone https://github.com/darkmice/dark-skill-forge.git
cd dark-skill-forge
```

把需要的 Skill 链接到 Agent Skills 全局发现目录：

```bash
mkdir -p ~/.agents/skills
ln -s "$PWD/focus-compass" ~/.agents/skills/focus-compass
ln -s "$PWD/ccpm" ~/.agents/skills/ccpm
ln -s "$PWD/dark-tribunal" ~/.agents/skills/dark-tribunal
```

已存在同名文件或目录时，先确认它的真实来源与改动，不要直接覆盖。客户端必须支持 Agent Skills 或扫描 `~/.agents/skills`；安装后通常需要新建或刷新会话。

## 新增 Skill

1. 在仓库根目录创建与 Skill 名称一致的目录；
2. 使用 Skill Creator 建立最小入口和必要投影；
3. 根据任务复杂度使用 CCPM 规范结构，并用 Dark Tribunal 审查行为变化；
4. 验证后提交到本仓库，再从全局发现目录建立链接。

## License

[MIT](LICENSE) © 2026 dark

---
**Dark Skill Forge · 暗黑技能熔炉**
把判断锻造成规范，把规范熔炼成可验证的交付。
[GitHub: darkmice/dark-skill-forge](https://github.com/darkmice/dark-skill-forge)
—— dark
