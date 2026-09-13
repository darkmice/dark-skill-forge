# Dark Skill Forge repository rules

## Source of truth

- This repository is the canonical source for skills maintained here.
- Put each skill in one root-level directory whose name matches the `name` in its `SKILL.md` frontmatter.
- Global discovery directories should link to these canonical directories. Do not maintain hand-edited duplicate installations.
- Existing standalone repositories are distribution mirrors. Author changes here first; do not silently edit a mirror and leave this repository stale.

## Creating or changing a skill

1. Read the target `SKILL.md`, its routed references, and all affected Agent projections before editing.
2. Use the available Skill Creator workflow for new or substantially revised skills.
3. Use CCPM when the skill has multiple consumers, duplicated rules, unclear authority, or cross-client projections. Do not force CCPM onto a confirmed single-owner local correction.
4. Use Dark Tribunal for `SKILL.md`, prompt, policy, Agent metadata, executable behavior, review, or release changes. Preserve read-only requests and existing authorization boundaries.
5. Keep the entrypoint concise. Put genuinely conditional detail in `references/`; do not add supporting files without a concrete consumer.

## Projection and compatibility rules

- Treat `SKILL.md` as the semantic authority. Agent-specific files under `agents/` are projections and must not broaden behavior or permissions.
- Keep shared `SKILL.md` frontmatter portable across Agent Skills clients. Put client-specific discovery and invocation settings under `agents/` unless the repository explicitly targets only one client.
- Relative links must resolve from their containing file. Companion skills must fail honestly when missing rather than being imitated from memory.
- Routing or loading another skill selects a method only. It never expands task scope, write permission, external side effects, or user consent.

## Validation and release

- For code work, define the validation matrix before implementation, but run routine test/check/lint/typecheck/build/E2E only after the current delivery stage has completed Dark Tribunal review and batched repairs. A stage is a user milestone or independently deliverable vertical slice, not a file, issue, or small fix.
- Do not rerun the same validation suite after each edit. If stage-end validation fails, batch the failures, repair them, close the affected review surface, then rerun failed checks and affected regression; rerun the full batch only when the impact expanded or cached evidence is invalid.
- Before review, run only a blocking minimal diagnostic when it is necessary to continue safely, and label it diagnostic-only rather than pass evidence. Explicit TDD requests, mandatory project gates, and immediate security-isolation evidence may override timing, not scope or honesty.
- Validate frontmatter and every changed YAML, TOML, script, and relative link with tools appropriate to the target client.
- Test meaningful behavior scenarios, including normal use, exclusions, conflicts, missing dependencies, authorization boundaries, and incomplete verification.
- Review from source to projections, then perform a second pass from consumer scenarios back to the source. Fix confirmed defects and repeat a closing pass after second-round changes.
- Scan public changes for credentials, personal data, internal-only paths, unfinished placeholders, generated junk, and nested repositories.
- Do not commit, push, publish, overwrite installations, or update standalone mirrors unless the user's current request authorizes that action.
