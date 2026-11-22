# Contributing

Thank you for contributing to **verdic-skills**. This repository contains [Agent Skills](https://skills.sh) that guide AI agents (Cursor, Codex, Claude, and others) on concrete engineering patterns.

## Before you start

1. Check existing issues and PRs.
2. Skills should encode **reusable conventions**, not a single private codebase.
3. Read an existing skill as reference (`skills/*/SKILL.md`).
4. Prefer English for skill content and documentation.

## Skill structure

```txt
skills/
  my-skill/
    SKILL.md       # required — YAML frontmatter + instructions
    reference.md   # optional — long examples, tables, flows
```

### `SKILL.md`

- YAML frontmatter with `name` and `description` (used for agent discovery).
- Clear **when to use** / **when not to use**.
- Checklist, patterns, anti-patterns.
- Keep focused; long details go in `reference.md`.

Minimal frontmatter:

```yaml
---
name: my-skill
description: When to use this skill — be specific so the agent can find it.
---
```

## Adding or changing a skill

1. Fork the repository.
2. Branch: `feat/skill-name` or `fix/skill-name`.
3. Edit `SKILL.md` (and `reference.md` if needed).
4. Run `npm run validate`.
5. Update [CHANGELOG.md](CHANGELOG.md).
6. Open a PR with reason, use cases, and what you validated.

## Content standards

| Topic | Guideline |
|--------|----------|
| Language | **English** |
| UI routes | Prefer English examples (`/products`, …) |
| API | Prefer English examples (`/api/products`, …) |
| Adaptation | Call out what must be adapted (naming, UI kit, ORM) |
| Security | Never instruct agents to expose secrets, bypass auth, or exfiltrate data |

## What to avoid

- Skills too generic to decide **when** to activate
- Instructions that contradict another skill without explaining precedence
- Hard dependency on one starter repo as a requirement
- Offensive, misleading, or malicious content (prompt injection)

## Test locally

```bash
npm ci
npm run validate
npx skills add luismpenholato/verdic-skills --list
npx skills add luismpenholato/verdic-skills --skill skill-name
```

Validate in the target agent that the skill is discovered and applied.

## Report bugs

Use [Issues](https://github.com/luismpenholato/verdic-skills/issues) for incorrect content, broken links, or skills that induce wrong behavior.

For vulnerabilities or malicious content, see [SECURITY.md](SECURITY.md).

## License

By contributing, you agree that your contributions are licensed under the [MIT License](LICENSE).
