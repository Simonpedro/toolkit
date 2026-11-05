# toolkit

Cross-app software-development practices for Simon's apps (Bille, Profe, Chatty, …) and the agents that build them. The goal is that a process that works well in one app becomes the standard for all of them.

## Structure

Each skill is a folder under `skills/` with a `SKILL.md`. The file opens with YAML frontmatter (`name` and `description`, where the description says when to use it). Optional supporting files sit next to it.

## Skills

| Skill | What it does |
|---|---|
| [`ux-complexity-gate`](skills/ux-complexity-gate/SKILL.md) | Checks that a screen (mockup or implemented) and the app's navigation are not too complex to use. |
| [`design-fidelity-gate`](skills/design-fidelity-gate/SKILL.md) | Checks that an implemented screen (web, Android, iOS) matches its mockup before merge. |

## Recommended order for a screen

1. `ux-complexity-gate` on the **mockup**.
2. Implement against the approved mockup.
3. `design-fidelity-gate` on the implementation.
4. `ux-complexity-gate` on the **implemented screen**.
5. Merge when both gates are green.
