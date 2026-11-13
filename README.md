# toolkit

A generic collection of software-development practices, packaged as skills for coding agents. Each skill turns a process that worked in real projects into steps, rules and thresholds an agent can follow and verify.

## Install

Uses the [skills CLI](https://github.com/vercel-labs/skills):

```bash
# All skills, globally, for Claude Code and Cursor
npx skills add Simonpedro/toolkit -g -a claude-code -a cursor -y

# A single skill
npx skills add Simonpedro/toolkit -g -a claude-code -a cursor -y --skill <name>

# List available skills
npx skills add Simonpedro/toolkit --list

# Update installed skills
npx skills update -g   # drop -g for project installs
```

Drop `-g` to install into the current project instead of globally.

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
5. Web only: a non-blocking [Modern Web Guidance](https://github.com/GoogleChrome/modern-web-guidance) review of the changes (accessibility, HTML/CSS over JS, performance, security).
6. Merge when both gates are green.

## Recommended external skills

| Skill | When to use it | Install |
|---|---|---|
| [Modern Web Guidance](https://github.com/GoogleChrome/modern-web-guidance) | Any web front-end work (not native apps): agents consult it while building UI, it reviews changes before merge, and `npx modern-web-guidance legacy-scan .` finds outdated patterns to modernize. | `npx skills add GoogleChrome/modern-web-guidance -g -a claude-code -a cursor -y` |
